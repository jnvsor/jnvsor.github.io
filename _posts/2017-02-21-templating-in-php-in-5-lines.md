Everything on the web needs templates.

While you can argue that MVC is overkill for simple applications, even the simplest of systems need templates as soon as they're outputting HTML.

Since PHP was designed for templating we can always use it as a quick 'n dirty templating system. Quick because you already have it and dirty because its unclear separation between code and template leaves it open for abuse.

While there are [many](http://platesphp.com/), [many](http://symfony.com/doc/current/templating/PHP.html) systems that exploit this property of PHP, I'd like to demo a simpler system that works in just 5 lines of code:

<script src="https://gist.github.com/jnvsor/55d7dee44d16a8e1edf5497ff2b517d8.js"></script>

## Line by line

So first we have our function declaration. We take a template as our argument, with an array of arguments to the template as an optional second argument.

Using [`extract`](http://php.net/manual/fr/function.extract.php) with `EXTR_SKIP` means we unpack the array into the current scope, and skip any array elements that already exist. This means we don't have to worry about overwriting superglobals.

By giving the template and args parameters long random names we can prevent accidental collisions there too. (Since `$args` and `$template` might be things you actually want to pass to a template) You can always make these longer if you're actually having collisions here.

We turn on output buffering and include the file in place. The file will be executed with the scope we're in, so the net effect is that all the args we passed to `template` will be available as variables in the included file.

In this example we just look for a file in a predefined folder to include, but if you want to add more logic and have multiple include folders to choose from that wouldn't take much code either.

Lastly we stop the output buffering and return it.

## What it can do

Here's a quick example of what it can do:

{% highlight php %}
<?php

include 'template.php';

define('TEMPLATE_DIR', __DIR__.'/templates/');

$args = [
    'title' => 'This web page',
    'text' => 'Hello world!',
    'logo' => 'mypic.svg',
];

echo template('myview.php', $args);
{% endhighlight %}

{% highlight php %}
<!DOCTYPE html>
<html>
    <head><?= template('head.php', ['title' => $title]) ?></head>
    <body>
        <header><?= template('header.php', ['logo' => $logo]) ?></header>
        <main><?= $text ?></main>
    </body>
</html>
{% endhighlight %}

{% highlight php %}
<title><?= $title ?></title>
{% endhighlight %}

{% highlight php %}
<img src="<?= $logo ?>" />
{% endhighlight %}

Needless to say, as it's just PHP you'll need to escape your output manually.

## With extends in 25 lines

Templates that can extend other templates are fairly popular. To do this we're going to wrap it in a class which will make it a bit more complicated, but shouldn't be too hard.

<script src="https://gist.github.com/jnvsor/cd4c9f3e811f6ac8c1f1cd9a3c9bbff4.js"></script>

So we've added a few features:

* When we instantiate our Template class we can pass in the template dir, so it's not a hardcoded define any more.
* We have a wrapper around our original template function which handles rendering extending parents.
* Because we're in a class we can tuck our args and template away inside the class to make the scope completely clean.

The new example code looks like this:

{% highlight php %}
<?php

include 'template.php';

$template = new Template(__DIR__.'/templates/');

$args = [
    'title' => 'This web page',
    'text' => 'Hello world!',
    'logo' => 'mypic.svg',
];

echo $template->render('myview.php', $args);
{% endhighlight %}

{% highlight php %}
<?php $this->extend('base.php'); ?>
<header><?= $this->render('header.php', ['logo' => $logo]) ?></header>
<main><?= $text ?></main>
{% endhighlight %}

{% highlight php %}
<!DOCTYPE html>
<html>
    <head><?= $this->render('head.php', ['title' => $title]) ?></head>
    <body><?= $contents ?></body>
</html>
{% endhighlight %}

{% highlight php %}
<title><?= $title ?></title>
{% endhighlight %}

{% highlight php %}
<img src="<?= $logo ?>" />
{% endhighlight %}
