---
title: Pushing to github pages without history
---

I was doing a small design change on the website and decided to write about a
very useful git alias I use for publishing my changes. It lets me use github
pages without having to have a complete history of the website on it.

Of course, someone can use the wayback machine or something to get my physical
address that was in my CV once upon a time, but you won't find it now by
spelunking the source code on github.

The trick is similar to my [git commit reset](/blog/2017/03/25/git-commit-reset/) post. Take the tree of the top commit and make it into a new commit. In the case of this site I make it an orphan commit so there's no history either.

<script src="https://gist.github.com/jnvsor/32890993d63615bbd8a6cde9f31c9dbe.js"></script>
