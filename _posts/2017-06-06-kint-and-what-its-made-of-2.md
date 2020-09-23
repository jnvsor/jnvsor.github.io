---
title: Kint, and what it's made of (part 2)
---

In [the last part](/blog/2017/05/14/kint-and-what-its-made-of/) I went through how Kint gets the modifiers and parameter names from the calling source file. The rest of the startup sequence is fairly boring so I'm going to skip straight ahead to the [`Kint_Parser`](https://github.com/kint-php/kint/blob/2.1/src/Parser.php)

## Initializing the parser

The parser is as close to "Good code" as it gets in Kint. I'm very proud of it. Additionally, because it's going to be called recursively many thousands of times it's an important target for performance optimization.

To initialize it you just construct it, and then call `addPlugin` with `Kint_Parser_Plugin` instances a few times.

While constructing you can pass it a maximum depth limit and a class name. This class name is the "Calling class" -- which has bearing on whether or not the caller has access to a private or protected property.

### Plugins, quickly

Plugins cause a significant amount of the runtime in the parser, even when they're not used. So the plugin system and the plugins themselves are the largest targets for performance improvements in the parser.

When we initialize our parser we call `addPlugin` on a few `Kint_Parser_Plugin` instances. `addPlugin` calls `setParser` on the plugin. Later the plugin will need the parser to halt the parse sequence or parse new child data.

Additionally, `addPlugin` adds the parser instance to a few slots in a multidimensional array indexed by applicable data types and triggers.

Triggers are class constants on `Kint_Parser` that represent different stages in parsing that a plugin can be called.

* `TRIGGER_BEGIN` is run before the parser touches the data, as soon as the type is known
* `TRIGGER_SUCCESS` is run after the value is parsed
* `TRIGGER_RECURSION` is run after parsing is aborted due to recursion
* `TRIGGER_DEPTH_LIMIT` is run after parsing is aborted due to hitting the depth limit

If a plugin returns `['array', 'integer']` when `getTypes` is called, it will be added under those types. `getTriggers` is similar except it uses a bitmask instead of strings from `gettype()`.

Because objects are always stored by reference this is relatively cheap on memory. More importantly looking up a prebuilt hash table helps cut down on the amount of plugins we have to loop through. Like I said in the beginning, this is code that will probably run several thousand times per page.

Even with the preloaded type and trigger filtering, most plugins won't apply to all data of that type. For performance reasons, all plugins in Kint start off with an `if` that causes an early return if the data doesn't match the type it's looking for. There's no sense going through the motions for parsing iterables if the object is a `DateTime`

### What we know and what we need to know

So now that we have our parser loaded up with plugins we have to decide: What do we want to know? First off, there are a few things we already know. These are metaproperties of the data we're parsing, not part of the data itself.

* What's the value name?
* What code do you need to access the value?
* How deep in the hierarchy is the value?
* What class is it in?
* Is it a public/private/protected property?
* Is it a reference?

For example we can't tell what a variable name is once it's been passed to a method. So we pass both the variable and a prebuilt `Kint_Object` filled with metadata to the parser.

When we're just calling the parser from the toplevel all we need to give it is the variable name and access path. See part 1 for how we get those.

### Parsing some basic values

Now we've been passed some data and a `Kint_Object` filled with metadata, we need to get the rest of it. Quite simply what we need to know now is:

* What type of data is it?
* What's the data?

Oh well that's easy! <small>&lt;/s&gt;</small>

Actually, the first one really is easy to answer. We just call `gettype()` on the data and store it in our `Kint_Object`.

After a `TRIGGER_BEGIN` plugin pass we use a switch based on the type to pick a method specialized to dealing with that data type.

Typically, a `Kint_Object_Representation` named `contents` will be added to the objects representation pool and set as its value representation, but resources and unknowns don't really have a value so they don't get a representation at all.

The simplest data types all use the same method. Ints, bools, nulls, and floats just store the data in the value representation and call it a day.

Strings, resources, arrays, objects, and 'unknown' get special treatment.

In the case of strings we additionally store the encoding and size, while resources get the resource type.

Afterwards the data and object are passed through a `TRIGGER_SUCCESS` plugin pass and returned.

### Values in values in values in values in values in values in values in values in values in values in values in values...

Since arrays and objects can contain other values, they're where things get complicated.

Using an array we loop through the array and recursively parse each element of the array. Before parsing we set the variable name, depth, access path, reference status, and other things we know from the parent context.

The reference status is simple -- if we make a (By value) copy of an array and change its contents and the original changes too, it was a reference. Of course we have to stash the value before we do this so we can put it back later.

Objects are mostly similar -- when you cast an object to an array you will get all the properties (Including the private/protected ones) in a strange format, where the keys are odd combinations of `\0`, the class name, and the property name.

We can also use reflection, which shows less information (No private properties) and can give problems with HHVM, so we use that in a second loop to catch anything missing from the first one (Notably string-integer properties).

#### Depth limit

When we parse an object or array that is above the depth limit given in the parser's constructor, we don't parse the children, instead we return early after a `TRIGGER_DEPTH_LIMIT` plugin pass.

We can ignore the depth limit on just about everything else, because only arrays and objects will increase the depth.

#### Recursion

Recursion is fun. We can't ignore it because that could result in outputting *far* more information than we want.

In the case of arrays, we take a random generated marker created in the constructor, and use it as a key on the array before parsing children. If we come across an array that has that marker set we know we've hit recursion and we return early after a `TRIGGER_RECURSOIN` plugin pass.

This means that when looping through the array we need to be very careful not to accidentally include the marker we've set for recursion.

In the case of objects (Which are always passed by reference) it's actually easier. We can use `spl_object_hash` to get a unique ID for the object instance, and store that in a big array on the parser. If we hit a hash that's already set, we're recursing.

The problem is that `spl_object_hash` only comes with PHP 5.3 and Kint supports 5.1. Luckily, we have a makeshift object hash in `var_dump` which gives us a number next to each object. We end up having to parse output buffered `var_dump` output to get it though, so this is inferior to `spl_object_hash`.

#### Access paths for the faint of heart

Access paths are amazing. They provide a one click copy paste expression that gets you exactly the value you want. (Though not always for writing)

The first step in access paths is to check if you actually have access. If not then showing an access path would be a waste of time and bytes. For arrays this is simple: If we have access to the array we have access to the child.

For objects this is a bit more complex. We need to know the calling context to see if we have access to individual private/protected properties. We pass this into the parser at construction based on the `debug_backtrace` output.

Once we have that we need to take the array key or property name and extend the current access path. If we're in an array the pattern to append will be something like `[$name]`, if we're in an object the pattern will be something like `->$name`.

#### Access paths for the stout of heart

The PHP docs give this helpful regex to validate variables names: `[a-zA-Z_\x7f-\xff][a-zA-Z0-9_\x7f-\xff]*`

Unfortunately, this is bullshit.

Variable variables allow you to use any binary sequence as a variable or property identifier. So when we're making our access paths we need to pay attention.

Access paths off of arrays can be easily implemented with `var_export` which will automatically escape nuls and other binary sequences which can't be represented in a simple string. This is great because we can just stick `'['.var_export($key, true).']'` on the end of the access path and voila -- we have our new access path.

With objects however we have to check the regex and then decide whether to add `'->'.$key` or `'->{'.var_export($key, true).'}'` depending on whether it's a valid variable string or not.

### Infinite gotchas

Of course just describing the principles behind parsing data wouldn't be complete without the relatively long list of exceptions upon exceptions upon exceptions.

#### Pass by value

Because the initial dump is pass-by-value, adding the recursion marker key to the toplevel array will have no effect. It will be recursed twice before hitting the recursion trigger.

Thanks to pass-by-value it actually *is* a separate array, and there's no way to fix this unless PHP somewhere adds a `get_func_args_ref` that also works with literals.

Kint 1 somewhat clumsily attempted to short-circuit this for clarity, but it didn't take into account users actually dumping a copy of an array (Leading to incorrect output)

In Kint 2 we just removed this "feature" entirely. It never worked right and as far as I can tell over the many months I've been thinking of it, it never will with current PHP implementations.

#### Altering data through references

Because user data may contain references and we're doing scary things like adding recursion markers to arrays we need to be very careful. If we were to say -- set all array elements to null when we loop through them, an awful lot of users would be complaining that Kint is causing random variables to empty.

This is worse with plugins which may be user-submitted, so the parser contains a helper method `getCleanArray`. The helper method takes the array by value (Creating a copy) and strips it of the recursion marker.

#### Impossible indices

As I've written before [there are impossible indices in PHP](/blog/2017/04/14/impossible-indices-in-PHP/). I'm not going to go into the specifics, but we basically need to detect these and alter the access paths when we hit one.

This usually involves the access path turning into `'array_values((array) '.$parent->access_path.')['.$i.']'` (Where `$i` is the numeric index of the value) for both arrays and objects.

Of course we also need to alter this detection based on whether or not we're on PHP 7.2 yet (Which fixes the impossible index issue)

#### HHVM

Ah good old HHVM. Where would we be without a language implementation that does everything just a little annoying bit differently?

In the parser, the issue of note is that HHVM actually has better type hinting than PHP expects. The aforementioned impossible indices are possible to get in HHVM but the reflection functions are typehinted for string properties. You'll get a nice fatal error if you use builtin reflection on one of those objects in HHVM.
