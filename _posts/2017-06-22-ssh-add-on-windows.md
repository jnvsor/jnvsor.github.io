---
title: ssh-add on windows
---

Why on earth would you want to use windows? I don't know, but I know that handling SSH on it is a pain.

Here's a script that makes `ssh-add` behave the same way in a bash shell on windows as it does on Linux. Run it manually once and it stays until you logout/shutdown. Add it to your `~/.bash_profile` and you're good to go.

<script src="https://gist.github.com/jnvsor/d7a97773223729f4cc867132f9274661.js"></script>
