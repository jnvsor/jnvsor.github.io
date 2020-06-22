---
title: Symfony with Apache rewrites
---

Symfony's HttpFoundation/HttpKernel is notorious for its unwillingness to be friends with Apache's mod_rewrite. While you can fairly easily rewrite into the `/web/` dir, pretty much everyone recommends using documentroot instead.

{% highlight apache %}
RewriteEngine On
RewriteRule (.*) /web/$1 [L]
{% endhighlight %}

The main problem is that when you rewrite with apache both the prefixed and non-prefixed urls will work. For static files this is ok, but if your public folder is named web and you want to add an actual `/web/` url to your app you're gonna have a bad time.

Links to the `/web/` page will show up at the index, and now all the links on the site will be prefixed with `/web/`. Eventually you can find your way back to where you were and go to `/web/web/` but that's hardly a nice solution.

The easy alternative however, is this little snippet here:

<script src="https://gist.github.com/jnvsor/3d2267b5774626f4cfc94c1d32bfe7ea.js"></script>

This extends the `Request` object and allows you to override the base url, so you can manually force it to the same base url as your rewrite. This will let you use Symfony and Symfony derived systems (Silex, Laravel, etc) both in subfolders and when you only have access to the public folder, without needing to alter virtualhost entries.
