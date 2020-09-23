Getting a [dynamic menu in Jekyll][jdm] is apparently really hard.

All the examples available either use ruby plugins, have the menu hardcoded into the `_config.yml` or just don't work. [Thinkshout made a good stab at it][thinkshout], wisely noting that urls provide an excelent hierachy for you to parse, but it doesn't quite cut it. The menu dissapears if you're on any page outside the menu, and a brief look brings to light a number of unhandled edge-cases.

This is the first time I've actually done anything with jekyll or liquid for that matter. I spent a whole weekend and 2 days after work learning the liquid idiosyncracies by trial and error and eventually stumbled upon a [menu that seems to work][jdm].

My menu is:

* Dynamic
* Hierachical
* Loads the menu items with classes indicating their state (branch/leaf, open/closed, active/inactive)
* Pure liquid - should work on any Jekyll site (Including github pages)

## The nitty gritty

I'm using my main include recursively, and another include as a boolean [function][hamish]. (With the slight difference from the linked article that I'm using jekyll's global variables instead of capturing the output as a string)

The main include:

{% highlight liquid %}{% raw %}
<ul class="menu">
{% for node in site.pages %}
  {% include menushow url=include.url node=node children=site.menu_show_all %}
  {% if retval %}
    {% assign branch = false %}
    {% assign open = false %}
    {% assign active = false %}
    {% for child_node in site.pages %}
      {% include menushow url=node.url node=child_node children=true %}
      {% if retval %}
        {% assign branch = true %}
        {% include menushow url=node.url node=child_node children=site.menu_show_all %}
        {% if retval %}
          {% assign open = true %}
        {% endif %}
        {% include menushow url=node.url node=child_node %}
        {% if retval %}
          {% assign active = true %}
        {% endif %}
      {% endif %}
    {% endfor %}
    <li class="{% if page.url == node.url %}selected {% elsif active %}active {% endif %}{% if branch %}branch {% if open %}open{% else %}closed{% endif %}{% else %}leaf{% endif %}"><a href='{{site.baseurl}}{{node.url}}'>{{node.title}}</a>
    {% if open %}{% include menulevel url=node.url %}{% endif %}</li>
  {% endif %}
{% endfor %}
</ul>
{% endraw %}{% endhighlight %}

The boolean function:

{% highlight liquid %}{% raw %}
{% assign retval = false %}

{% assign levelpath = include.url | split: '/'  %}
{% assign filename = include.url | split: '' | last %}
{% if filename != '/' %}
  {% assign filename = levelpath | last | truncate: 6, '' %}
  {% if filename == 'index.' %}
      {% assign levelpath = levelpath | pop %}
  {% endif %}
{% endif %}
{% assign levelpath = levelpath | join: '/' | append: '/' %}

{% assign nodedepth = include.node.url | split: '/' | size | minus: 1 %}
{% assign filename = include.node.url | split: '' | last %}
{% if filename != '/' %}
  {% assign filename = include.node.url | split: '/' | last | truncate: 6, '' %}
  {% if filename == 'index.' %}
    {% assign nodedepth = nodedepth | minus: 1 %}
  {% endif %}
{% endif %}
{% if nodedepth < 1 %}{% assign nodedepth = 1 %}{% endif %}
{% assign thisdepth = levelpath | split: '/' | size %}
{% if include.url == '' %}{% assign thisdepth = 1 %}{% endif %}

{% assign baselength = levelpath | size %}
{% assign nodebase = include.node.url | truncate: baselength, '' %}
{% assign pagebase = page.url | truncate: baselength, '' %}
{% if include.children %}
  {% assign pagebase = levelpath %}
{% endif %}

{% if include.node.title and nodebase == levelpath and pagebase == levelpath and nodedepth == thisdepth %}
  {% assign retval = true %}
{% endif %}

{% endraw %}{% endhighlight %}

Starting right off the bat we've got our menu and our loop through the `site.pages`, and our function include:

{% highlight liquid %}{% raw %}
<ul class="menu">
{% for node in site.pages %}
  {% include menushow url=include.url node=node children=site.menu_show_all %}
  {% if retval %}
{% endraw %}{% endhighlight %}

The include sets the `retval` global variable to either `true` or `false` depending on whether or not the link in question should be shown at this point in the menu. We pass in the page in question and the current `include.url` which acts as both a recursion depth check and a location. (So we don't display menu items at the same depth but under a different tree)

Looking at the function include, we first set the global to `false` - all these variables are global and we don't want a `true` from the previous iteration gumming up the works:

{% highlight liquid %}{% raw %}
{% assign retval = false %}
{% endraw %}{% endhighlight %}

The next mouthful checks the current recursion url for an index file then strips it to make the matching easier. (`/folder/index.html` is basically the same as `/folder/`)

{% highlight liquid %}{% raw %}
{% assign levelpath = include.url | split: '/'  %}
{% assign filename = include.url | split: '' | last %}
{% if filename != '/' %}
  {% assign filename = levelpath | last | truncate: 6, '' %}
  {% if filename == 'index.' %}
      {% assign levelpath = levelpath | pop %}
  {% endif %}
{% endif %}
{% assign levelpath = levelpath | join: '/' | append: '/' %}
{% endraw %}{% endhighlight %}

* I'd split the filename by `.` but in the first iteration there's a `nil` after the first split and jekyll will refuse to split a `nil`.
* I'm not worried about not having slashes at the end, because without the slashes jekyll spawns an extensionless file and my browser tries to download it - the menu won't show up right on that page, but neither will the rest of the page so you've got a bigger problem!
* I only check the name of the file because it's entirely possible you might want jekyll to generate PHP files (Or some other extension)
* Liquid's oh-so-wonderful consistency means that `'/' | split: '/' | size` is 0 and `'/a' | split: '/' | size` is 2! This is one of 2 reasons we append `'/'` at the end.
* > What happens when you have a folder called `index.*`?

  Well then you don't have an *actual* index file in this folder (Conflicting names) and it can't possibly go any deeper anyway. Besides before you get to the index check it already checks to see if the last character is a `'/'`.

Here we do the same thing to the node url, but now we're just doing it to get the node's depth. We get the recursion depth while we're at it:

{% highlight liquid %}{% raw %}
{% assign nodedepth = include.node.url | split: '/' | size | minus: 1 %}
{% assign filename = include.node.url | split: '' | last %}
{% if filename != '/' %}
  {% assign filename = include.node.url | split: '/' | last | truncate: 6, '' %}
  {% if filename == 'index.' %}
    {% assign nodedepth = nodedepth | minus: 1 %}
  {% endif %}
{% endif %}
{% if nodedepth < 1 %}{% assign nodedepth = 1 %}{% endif %}
{% assign thisdepth = levelpath | split: '/' | size %}
{% if include.url == '' %}{% assign thisdepth = 1 %}{% endif %}
{% endraw %}{% endhighlight %}


We manually assign `nodedepth` to 1 if it's below it. 1 is the depth of our `base_url`.

We do the same to `thisdepth` if the recursion url is `''`. If we did `'/'` we'd have the same problem as with `'/index.html'` where the index file is a child of itself and jekyll freaks out with a recursion depth error.

This way when jekyll iterates to `'/'` and tries to find children the depth of 0 will result in none found. The root index is quite the odd exception.

The extra `minus` on `nodedepth` is an artifact of the fact that `include.node` is a subpage (IE: `/sub/page/`) while `include.url` is it's parent (IE: `/sub/`)

Now we calculate the size of the `levelpath` (Which you might remember is the cleaned-up `include.url` AKA recursion depth) and truncate the current page and current node urls so we can check for equality:

{% highlight liquid %}{% raw %}
{% assign baselength = levelpath | size %}
{% assign nodebase = include.node.url | truncate: baselength, '' %}
{% assign pagebase = page.url | truncate: baselength, '' %}
{% if include.children %}
  {% assign pagebase = levelpath %}
{% endif %}
{% endraw %}{% endhighlight %}

This is why we appended the `/` to `levelpath` at the start - without that we might match `/sub2/page.html` as being a child of `/sub` since they both start the same.

The `include.children` variable basically makes the function ignore the current page url in all this, but we'll go into more depth on that later.

Finally the if statement:

{% highlight liquid %}{% raw %}
{% if include.node.title and nodebase == levelpath and pagebase == levelpath and nodedepth == thisdepth %}
  {% assign retval = true %}
{% endif %}
{% endraw %}{% endhighlight %}

* The standard check for a title is there
* Then we check whether the node path starts with the recursion path. (Whether the node is under the recursion path)
* Next we check whether the page is under the recursion path so we don't open hierarchies we're not currently visiting.
* Lastly we check that the depth of the item is correct so we don't get any further nested pages.

If all these pass we set `retval` to `true`!

## The less gritty

Now that we're done with the hard part we can get back to the actual menu.

We've established that this item needs to go here, now we need to establish what type it is.

As always we start off by assigning the variables to `false` so holdovers from the last iteration don't surprise us:

{% highlight liquid %}{% raw %}
<ul class="menu">
{% for node in site.pages %}
  {% include menushow url=include.url node=node children=site.menu_show_all %}
  {% if retval %}
    {% assign branch = false %}
    {% assign open = false %}
    {% assign active = false %}
{% endraw %}{% endhighlight %}

So we start looping through the sub pages to find out if there are any by passing the `node.url` and `child_node` instead of the `include.url` and the `node`:

{% highlight liquid %}{% raw %}
{% for child_node in site.pages %}
  {% include menushow url=node.url node=child_node children=true %}
  {% if retval %}
    {% assign branch = true %}
{% endraw %}{% endhighlight %}

Surprise! We're going back to the function include!

Remember this? What the `include.children` parameter determines is whether or not the viewer has to be viewing a page in the hierarchy to see the items.

{% highlight liquid %}{% raw %}
{% assign pagebase = page.url | truncate: baselength, '' %}
{% if include.children %}
  {% assign pagebase = levelpath %}
{% endif %}
{% endraw %}{% endhighlight %}

When we set this to `true` we can do several interesting things:

* Find out whether we *would* show menu items if we *were* on that page, as seen in the subloop:

{% highlight liquid %}{% raw %}
{% for child_node in site.pages %}
  {% include menushow url=node.url node=child_node children=true %}
  {% if retval %}
    {% assign branch = true %}
{% endraw %}{% endhighlight %}

* Show all menu items even when they are outside our current path based on a config variable, as seen in the first include:

{% highlight liquid %}{% raw %}
{% include menushow url=include.url node=node children=site.menu_show_all %}
{% endraw %}{% endhighlight %}

We know this menu item (`node`) will be shown, and we know it has sub items (`branch`) but we don't know whether it is open yet.

If it's open we're going to add a class to the `li` so we can theme it appropriately, and (more importantly) we can decide whether or not to recurse and generate another menu.

We could use the global variable `levelpath` for a simple equality check but taking that out of the function include feels even more icky than writing this thing in liquid, and recreating `levelpath`, `baselength` and `pagebase` just for an equality check feels ickier still.

DRY: We have a function that checks those already! Sure it's doing the depth and node path checks twice over but we're already looping over way too many nodes because liquid doesn't have a break statment anyway:

{% highlight liquid %}{% raw %}
{% include menushow url=node.url node=child_node children=site.menu_show_all %}
{% if retval %}
  {% assign open = true %}
{% endif %}
{% endraw %}{% endhighlight %}

Lastly we throw performance to the wind and do it again to determine whether this node is active - notably we leave `children` off of this since even with `site.menu_show_all` we don't want active classes going on open branches that aren't in this path.

{% highlight liquid %}{% raw %}
{% include menushow url=node.url node=child_node %}
{% if retval %}
  {% assign active = true %}
{% endif %}
{% endif %}
{% endfor %}
{% endraw %}{% endhighlight %}

We finish off by *finally* rendering the menu item complete with classes, and recursing to the next level with the `include.url` set to `node.url`:

{% highlight liquid %}{% raw %}
<li class="{% if page.url == node.url %}selected {% elsif branch and open %}active {% endif %}{% if branch %}branch {% if open %}open{% else %}closed{% endif %}{% else %}leaf{% endif %}"><a href='{{site.baseurl}}{{node.url}}'>{{node.title}}</a>
{% if branch %}{% include menulevel url=node.url %}{% endif %}</li>
{% endif %}
{% endfor %}
</ul>
{% endraw %}{% endhighlight %}

We finish the loop and finish the menu.

And *that* is why it took 5 days of trial and error :P


[jdm]: http://jnvsor.github.io/jekyll-dynamic-menu/about/
[thinkshout]: http://thinkshout.com/blog/2014/12/creating-dynamic-menus-in-jekyll/
[hamish]: http://hamishwillee.github.io/2014/11/13/jekyll-includes-are-functions/
