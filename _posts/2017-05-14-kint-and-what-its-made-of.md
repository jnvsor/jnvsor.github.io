---
title: Kint, and what it's made of (part 1)
---

I released version 2.1 of Kint this weekend, and I decided to write a bit about some of the more interesting internals I've run into in the last year or so of rewriting it.

## The source parser

Kint has a number of "Black magic" features which require parsing the source code it was called from.

* Parameter names

    You need to know what parameter names are to print them in the output (Duh) but you also need them to create access paths.
* Modifiers

    You need to parse the source code to find the modifiers Kint was called with. For example, while `d($var)` will dump a variable, `+d($var)` will disable the depth limit and dump a variable.

### Debug backtrace is your friend!

* Tells you the line and file Kint was called from. This will be necessary to accomplish the two points above
* Tells you the context (Class) Kint was called from. This is necessary to determine whether you have access to certain private or protected properties

### How it works

Kint contains a [`Kint_SourceParser`](https://github.com/kint-php/kint/blob/2.1/src/SourceParser.php) single-purpose class which takes source code, a line number, and a callable, and tries to pull out information about all the calls it can find.

Since more than one statement can exist on one line, the callable has some extra restrictions.

* We can't tell if a variable in source code is a specific instance, so normal method calls are off limits.

    `$object1->dump(); $object2->dump();`
* We can't tell if a variable in source code is a specific closure, so closures are off limits too.

    `$closure1(); $closure2();`
* We could parse the entire class, namespaces included, aliases included, but we don't because we're lazy.

Long story short: We only accept static method calls and function calls, and then we only parse the final part of the class, not the namespaces.

We use [`token_get_all`](http://php.net/manual/en/function.token-get-all.php) on the source code so PHP lexes it for us and all we have to do is loop through tokens.

#### Finding the function call

Looping through tokens we wait until we find a `T_STRING` that's the same name as our callable function/method and backtrack to make sure the class is correct (or it doesn't have a class, in the case of a function call)

I actually changed this recently. Now the class will store the last 3 non-whitespace tokens in a separate array specifically for this purpose. [PHP7 arrays have excellent data locality](http://nikic.github.io/2014/12/22/PHPs-new-hashtable-implementation.html), and it seems that backtracking past whitespace to the last "real" token killed performance.

#### Getting the parameters

Anyway, once we know the callable is the right one, we start actually parsing. We have a list of tokens that increase and decrease "Depth" and simply keep going until we hit a comma or a close paren at the base level. This means we either need to start storing a new parameter or we're completely done.

While we're doing this, we store the parameter tokens in one array, and deliberately throw away anything below the toplevel, and store this in a second array. This lets us change parameters like `$var['really long key']` into `$var[...]` for display.

#### Checking the line number

At the start of the main loop we take the string of the token and count the number of newlines in it. This is our line number.

Even if `token_get_all` supported line numbers as far back as 5.1, it only shows the starting line number, so if you had a string token (Like a comma) right after a `T_WHITESPACE` with a number of newlines, we wouldn't have the right line number.

In any case, after we've hit the close parenthesis we check our line number. If our line number is still below the line shown in the backtrace, we've been parsing an earlier call than the one we want, so we just continue.

If the final line number is equal or greater than the backtrace line number, the parentheses of the function call cover the line in question and this may be the one we're looking for, so we continue parsing and save it.

You might think you could just start at the line and work backwards to the first semicolon token you see before parsing, but this won't work either. If you dump a closure in place you can have actual semicolon tokens.

In short, you can take any file and wrap it in `d(function(){})` and it will be valid PHP, so no matter what you have to parse from the very beginning.

#### Handling expressions

What do we do if we're passed an expression? Kint can dump the value of an expression no problem, but the access paths are a more complicated question. `$array1 + $array2[1]` is something entirely different to `($array1 + $array2)[1]`, and we need to tell which to use in access paths when someone calls `d($array1 + $array2)`.

Luckily, the short parameter version has already done most of the work for us. By removing all but the toplevel tokens, we can quickly scan the short parameter for a list of operator tokens and if it contains one we simply mark it as an expression and wrap it in parentheses later.

#### Getting modifiers

Getting the modifiers is relatively simple. We just loop back through the tokens to the start of the call and look for them. Since this is only done once we've confirmed the function call matches, it's unlikely to cause a huge slowdown.

#### Continuing the parse

We've parsed a function calls parameters, with the right function name, on the right line. You'd think we're done right?

But what happens when there are multiple dumps on one line? Worse, what happens when there are dumps *inside each other*?

`d($var1); d($var2);`

`d(d($var1));`

Long story short, we've finished parsing the parameters belonging to the hit, but we need to continue parsing until we hit the next line, since it's possible there are more function calls to be found. In the first case the class will find a pair of function calls with parameters `$var1` and `$var2`, in the second it will find a pair of function calls with parameters `d($var1)` and `$var1`.

## Out of the source parser

Once the source parser has run we have a number of function calls with parameters, and we have to pick which one is the one we need. This happens back in the [`Kint` class](https://github.com/kint-php/kint/blob/2.1/src/Kint.php#L487)

### Counting parameters

The easiest way to tell is by counting the parameters. If Kint was called with 3 parameters, and only one of the matching calls has 3 parameters, it's easy to pick out the ones that don't match.

### T_ELLIPSIS, bringer of woe

But PHP 5.6 added a new feature, one which makes counting parameters awfully annoying. The `T_ELLIPSIS` token allows you to unpack an array as arguments in place, instead of relying on `call_user_func_array`.

This gives us a few more things we have to check.

If the last parameter begins with an ellipsis we can automatically generate as many parameters as we need to fill parameters. So something like `d($a, ...$b)` with 3 parameters would result in `$a`, `array_values($b)[0]`, `array_values($b)[1]`.

If there are more than one ellipsis, we can't tell when they switch from one to the other, so we just give up. `d($a, ...$b, ...$c)` might have only 2 parameters, and we can't tell if the second is from `$b` or `$c`, so we only record `$a`.

### Fallback

If we don't have the parameter names, we replace the parameters with numeric ones, so that we can at least tell which is which in the access path.

If we have 3 parameters and multiple function calls, we'll end up showing `$0`, `$1`, `$2`. Since variables can't begin with a number unless you're using dynamic variable naming, it should be clear where in your access path you should put the variable you're dumping.

Similarly, if we have multiple ellipses we'll end up showing `$a`, `$1`, `$2`.

#### Modifiers

Unfortunately, if we can't distinguish the function call there's nothing to do about the modifiers. They've been lost either way.

Note that none of these counting tricks are necessary if the end user doesn't do weird things like dump inside a dump, or dump on multiple lines, so if we only have one function call we can just take that.

## Everything else

This whole page has been just about getting the parameter names and modifiers from the calling source code file. I haven't touched on the actual parsing and rendering of the data. That's for part 2.
