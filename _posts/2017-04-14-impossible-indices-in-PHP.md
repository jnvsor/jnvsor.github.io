---
title: "Impossible Indices in PHP"
---

It's possible to have an impossible index in PHP. In fact, it's possible to have an impossible property on an object too.

When you add to an array in PHP it will either add it as a string key, or as an integer key. If you add an integer string as a key (IE: `ctype_digit` and doesn't start with `0`), PHP will internally cast it to an integer. This always works whether it's added by literal or by variable, whether it's reading or writing.

{% highlight php %}
<?php
$array = [
    'key' => 'value',
    '1234' => 'value',
];
var_dump($array);
{% endhighlight %}

<pre>
array(2) {
  ["key"]=>
  string(5) "value"
  [1234]=>
  string(5) "value"
}
</pre>

Note the lack of quotes around the second key.

## Impossible variables in PHP

Did you know you can create variables and properties that aren't supposed to be legal? According to [the PHP docs](http://php.net/manual/en/language.variables.basics.php) valid variables and properties are supposed to match the regex `/[a-zA-Z_\x7f-\xff][a-zA-Z0-9_\x7f-\xff]*/`.

In practice this is only the requirement for variables named directly. With variable variables you can name a variable after any blob you want. `${""}`, `${"\0"}`, `${file_get_contents("windows_10.iso")}` are all valid, and variable length is limited more by your available memory than by design.

Since objects are implemented internally much the same way as arrays, it makes sense that you can add integer indices as dynamic properties.

{% highlight php %}
<?php
$object = new stdClass();
$object->{'key'} = 'value';
$object->{1234} = 'value';
var_dump($object);
{% endhighlight %}

<pre>
object(stdClass)#1 (2) {
  ["key"]=>
  string(5) "value"
  ["1234"]=>
  string(5) "value"
}
</pre>

Hmm. Looks like objects do some casting too! Since objects can only have string properties, integers are automatically cast to strings.

## Black magic casting <small>monkey patching on drugs</small>

So what happens when we cast the array to the object, and vice-versa?

{% highlight php %}
<?php
$object = new stdClass();
$object->{1234} = 'value';
$array = (array) $object;
$array[1234] = 'value';
$object = (object) $array;

var_dump($array);
var_dump($object);
{% endhighlight %}

<pre>
array(2) {
  ["1234"]=>
  string(5) "value"
  [1234]=>
  string(5) "value"
}
object(stdClass)#2 (2) {
  ["1234"]=>
  string(5) "value"
  [1234]=>
  string(5) "value"
}
</pre>

We have performed the forbidden ritual, and now an unnatural beast is born!

We have an array with a numeric string key, and an object with an integer property. It's impossible to get at these values without either iteration or casting them back to their original states.

When you iterate over them you will be able to tell the difference because the key will be either a string or an integer, and you can alter them if you take the value by reference, but you can't unset the impossible indices because it will automatically cast you to the wrong type when you try.

Of course, by the time anyone reads this PHP 7.2 will be out. In 7.2 the indices will be cast when the variable is cast, so this will never happen again. Oh well!
