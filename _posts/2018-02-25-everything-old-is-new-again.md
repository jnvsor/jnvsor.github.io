---
title: Everything old is new again
---

Over the years we've had many techniques for linking CSS. Here's a small list.

## Inline CSS

Applying CSS rules to DOM elements directly through the style attribute. Simple and functional.

## Style tags

Now we can bundle query selector based blocks of CSS rules for many elements at once through a style tag. Simple and functional.

Inline CSS is now bad because it's rigid.

## CSS files

Now we have separate files that let us cache the contents of a style tag separately from the page. Simple and functional.

Style tags are now bad because they can't be changed in a single location.

## Multiple CSS files

Now we want to lower the bandwidth used accessing our website, so we split the CSS into separate files that can be loaded conditionally depending on whether they're used on the page or not. Simple and functional.

Single CSS files are now bad because they waste bandwidth.

## Compiled single CSS files

Now that bandwidth is cheap, we want to reduce load times by lowering the number of requests to our server, so we compile multiple CSS files into a single file. Simple and functional.

Multiple CSS files are now bad because they increase the number of requests.

## "Google PageSpeed said so"

Now that the marketing department knows about Google PageSpeed, we have to split our CSS up into "Crucial" css that goes into a style tag "above the fold" and non-crucial CSS that goes into a non-blocking separate file. We hope no-one notices the 2-3 seconds during first load that the site looks like shit and jumps around before the CSS file loads. Simple and functional.

Compiled single CSS files are now bad because Google said so and marketing believes them.

## HTTP2 multiplexing

Now that HTTP2 can send multiple files in a single request, we want to save bandwidth again and split the CSS into multiple files, plus the style tag for the crucial CSS. Simple and functional.

Google PageSpeed is now bad because Google admitted it gave horrible advice.

## The best way to link CSS files

Now that we've been doing this for 15 years we have multiple CSS files compiled into one on the fly and cached based on a hash of the concatenated filenames, so we end up loading a different large CSS file with duplicate content for every page request which starts with a broken page because of the half-working above the fold styles combined with inline style attributes on some DOM nodes. Simple and functional.

HTTP2 is now bad because we couldn't find a server that supported PHP3 and HTTP2 at the same time.
