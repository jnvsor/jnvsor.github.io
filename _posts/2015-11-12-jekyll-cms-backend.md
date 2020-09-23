---
published: false
---

A lot of our customers at work say they want a custom themed CMS, then they use the same less than 10 static pages for half a decade, only to call us when they want the text tweaked, menu rearranged, or theme updated.

In a situation like this my first instinct would naturally be to replace these things in Jekyll or something more fitting such a small scale.

Of course, the customer explicitly says they want a CMS - even if they really don't. It's a big selling point, so there has to be some way for the user to edit the Jekyll site.

Now we could just give them FTP access and a big red button that says "Regenerate site" but users being users this is unlikely to work. So I'm going to see about working on the neccesary subparts to make something like this work.

### A YAML editor

While this isn't strictly neccesary, a nice point and click YAML editor with a fancy javascript UI would go a long way towards making configuration of websites easy.

A variation on this could be used to create a YAML menu hierarchy giving the user control over the menu.

### A text editor

Markdown is great, but even better (For users) is CKEditor. If there's even a small chance to get a decent editor with markdown support, I'll take it.

Needless to say, the normal interface will be very dumbed down - pages and posts, CKEditor input, and a fixed set of YAML options.

### A file browser/editor

The more advanced but still-too-clueless-to-use-an-FTP-client users might want to edit the files by hand. In this case plain text is more than enough, but experience says this will be hard to get right - file managers are notoriously slow.

### A login system

A small database backend to handle logins and settings, along with storing some backups and sane defaults so the user can fix their site when they inevitably screw it up - would be the final touch for this system.

### And one more thing...

Most customers want that one little bit of dynamic website that screws with this whole plan: A contact form.

This is usually done server side to shield the email address in question from an onslaught of botspam due to the emailaddress being public. Unfortunately, Jekyll is a bit of a pain with regards to this, and flat out refuses to use clean urls with a php extension - instead opting for some strange bastardization like `index.html.php`

[I wrote a quick 'n dirty module to fix that.](https://github.com/jnvsor/jekyll-php)
