---
published: false
---

For those of you not acquianted with [Kint](http://raveren.github.io/kint/), it's a wonderful PHP debug library.

There have been many of these over the years starting with [Krumo](http://krumo.sourceforge.net/) (Now abandoned) but Kint is currently the most commonly used one with wrappers for numerous frameworks.

## A bit of ancient history

Back in the paleolithic era of 2014 I was doing a lot of work in [Drupal](https://www.drupal.org/). While called a CMS because of it's fervent preference of a mouse over a keyboard, Drupal is well and truly in the framework category.

As any Drupal developer knows, if you're doing any custom code you want some decent debug output, and the [Devel](https://www.drupal.org/project/devel) module is it for Drupal.

Back in the paleolithic era using this module on a daily basis was normal. Not for any of the Devel specific features (Except the occasional dummy content generation) but because it provided automatic bindings for Krumo.

This was fantastic. Devel even had some custom code in their fork of Krumo to print the code you would need to access a specific variable.

But all things must come to an end, and with the now long-past Krumo abandonment, Devel looked to other options to provide it's debug output.

## A bit of recent history

I haven't done anything major in Drupal in a long time but I *have* been doing from-scratch projects. Without fail whenever I needed a debugger Kint was the one I turned to.

In a composer project it was readily available from the moment I required `autoload.php`, and in any other I simply required it in the index and it was ready to go.

I started submitting PRs like bug fixes and [an implementation of the access path code](https://github.com/raveren/kint/pull/177#issuecomment-172320222) from Devel's Krumo fork I liked so much.

As such I may be considered biased in what I'm about to say by superior familiarity with Kint internals but I think it's all valid nonetheless.

## The Devel Kint situation

The Devel module has an *old* version of Kint. Think *more than 2 years old*. Despite this Kint has been getting increasing numbers of bug reports from Devel users and developers claiming things are broken when they're not.

Just looking at the list of open bugs in Devel with the word "kint" in them immediately shows a long list of stuff that has been fixed in Kint for years, or (like the majority) is a Devel implementation detail. Worse, loads of these then proceed to blame upstream.

1. The bug report that spilled [out of Devel](https://www.drupal.org/node/2643392) and [into Kint](https://github.com/raveren/kint/issues/193) prompting this blog post:

    **Kint shows the file and line from devel module, not from where dpm() was called**

    In the version of Kint that's *not* 2 years old, simply adding your wrapper function to `\Kint::$aliases` will do the trick.

    One of the Devel devs showed up and complained that this would break the wrapper function given that they use the unary modifiers to ensure output returns instead of echoes:

    > The big downside of use \Kint::$aliases is that you loose the modifiers functionality and for this reason it is not used in devel [..] kint [echoes] the variable instead of return the output

    In the version of Kint that's *not* 2 years old, you can simply change this via the `\Kint::$returnOutput` boolean. Of course this still begs the question why they didn't just use output buffering if it was such a big problem.

    *Why would anyone think this is a bug in Kint?* This is clearly a Devel issue. If it wasn't, the bug report would be about something entirely different. (Perhaps "Feature request: Adjust mini stack trace source" if it wasn't already in Kint)

2. **kint() function not working in php 7**

    To quote by a Devel dev:

    > [..] the problem seems to be related to the kint library not with devel module [..] seems that with php 7 kint has some problems with call_user_func_array(); when call_user_func_array is used kint wrongly detects the modifiers and detect @ even when there is no one

    This is bullshit plain and simple. Neither the current *nor the 2 year old version* have this bug in PHP7. This is 100% Devel's fault, and the dev happily plugs his fingers in his ears and blames upstream.

3. **Tame Kint's output and make it actually useful**

    Because Kint isn't useful at all right?

    Here I'm less talking about code and more flaming about the bullshit I've spent a few hours reading on the Devel bug tracker.

    > Do you see all that wasted space? Why is it not giving me any hint about what's inside the array (in addition to the count)?

    If you wanted a mini dump of the contents right there on the bar you'd need it to be fairly large as there's no guarantee of the size of the output container. Combine this with the fact that it's recursive and you'd absolutely *flatten* any page requests.

    A Kint backtrace from inside a twig template on a default [bolt](http://bolt.cm) install is already so detailed it produces 75 **megabytes** of source code. With this suggestion that would go to something closer to 100 megabytes.

    > Let's open one array — WHAM! [..] Kint expands everything fully and we're hit with much more than we usually care to see

    If you press the little plus box yes, you will see more than you care to see. That button unfolds everything recursively. With the aforementioned backtrace unfolding one of the objects recursively takes *9 seconds* in what is not particularly slow javascript, and has been run through google's [closure compiler](https://developers.google.com/closure/compiler/).

    If you just click the bar (Which is much easier to hit anyway) it will just open one level - instantly.

    > Now it looks almost as good as Krumo

    Ok I'm just flat out calling bullshit on this one, and I didn't have to look far to prove it.

    One of the other open Devel issues contains [this picture](http://monosnap.com/image/NDKaUVXnORwKuxBWMdk2bZfrAnzLnd#) of Devel Kint and Krumo output next to eachother.

    The Krumo output is slightly larger than the Kint output, and with more empty space. Additionally Kint conveys more information than Krumo:

    * The variable name
    * The called function name
    * A small unfoldable backtrace (Maybe the Krumo one could be unfolded too - I don't remember)

    > And there's lots of wasted space again! Who cares about the namespaces at this point?

    It's a **debug library**. If it were to willy nilly disable namespace output you'd be bitching on the Kint bug tracker instead of the Devel one.

    > And who needs to be told that these are "object"s?

    2 year old code.

    > Or even that they have 5 top-level members.

    You were *just* complaining about the lack of information about it's contents.

    > The only interesting thing for 95% of the people (wild guess!) looking at a TranslatableMarkup object is the string inside. We're lucky that it's the first member in the object, it could very well be two browser pages down.

    Absolutely true. This is why Kint has custom parsers that allow it to render different types specifically. (Though looking at it now that system could use some updating to make it more namespace-freindly)

## tl;dr

I'm flaming because the Devel devs and users have an old, outdated version of Kint and they're blaming Kint upstream for their poorly written wrapper.
