---
title: Your input is bad, and you should feel bad
---

Things that screw up your input are bad. Things that screw up your input on purpose are worse.

## wORDpRESS

You've heard of wordpress, the world's most popular remote execution engine (With a blog attatched)

wordpress is widely known to be the primary source for PHP hate in the industry, with a combination of bad ideas, bad planning, and bad implementation providing a trifecta of stupid shit for people to take easy shots at. That's what I'm going to do here.

Specifically, wordpress has a tendancy to screw with your input in multiple ways. Any time you type "wordpress" into wordpress, it will automatically capitalize itself for you. Helpful.

Gotta love a content management system that puts it's brand's capitalization above it's users ability to input content. This will mess up any URL you attempt to link that doesn't adhere to wordpress' arbitrary capitalization requests.

The filter in question that causes this is `capital_P_dangit` which is infamous because of what it does, the fact that it was added without any community consultation, the fact that it's still there, and the fact it's preceded with a comment laughing about how the function name breaks wordpress' coding standards. (They had those? Wow...)

## MySQL

MySQL and it's successor MariaDB are weakly typed. This gives them the same advantage as weakly typed programming languages: They're great for RAD. You can dump some data in there and it will just work. Until it doesn't.

Weakly typed values may be fine in a programming language, but a database's sole purpose is to keep data consistant. The easiest way to do that is to error on invalid inputs like every other relational database I've heard of. Instead you get this:

```SQL
CREATE TABLE `test` (`switch` set('0','1'));

INSERT INTO `test` VALUES (1);

SELECT * FROM test;

+--------+
| switch |
+--------+
| 0      |
+--------+
```

I guess that says it all doesn't it?

## Sublime Text

Before I switched to sublime text I had the pleasure of programming in Geany. Geany is a great text editor that comes as a default on bunsenlabs (Crunchbang's successor) and handled my programming tasks for years.

When I switched to ST3 I stayed for the project-wide fuzzy symbol search, not for the basic text editing abilities. Actually the basic text editing abilities are pretty bad.

Actually the basic text editing abilities have me wondering whether I should jump ship before this editor decides to `rm -rf /` or something. In most text editors, when you paste in stuff, your editor doesn't automagically change what you pasted in. Not so in sublime.

You see, despite being one of if not the most popular programming text editor out there, sublime manages to screw up one of the most important features of a programming text editor: Indentation.

Indentation with spaces is the most popular form of indentation out there (Whether you're a fan of it or not) and the way ST3 implements it is to simply blindly replace any input tabs with spaces. It's literally impossible to get a tab into a file in ST3 without switching to tab indentation mode.

This even affects pastes. A common workflow for me is to paste a table into a text editor and use regex or other niceties to turn it into a query or something similar. You can't do this in ST3. It will take your paste, tie it to a water pump, and beat it to death with a baseball bat until it forgets tabs ever existed.

Your best option then is to open a new file, switch to tab indentation mode, paste it in, and stay in tab mode while you do your manupulations unless you want it to replace all the tabs on the line where you did a single character regex replace.

This aggrivated me so much that my only options were to go back to Geany and give up on fuzzy search, or write a plugin to fix it. [So I wrote a plugin to fix it.][1]

ST3 still doesn't approach the basic text-editing sanity of Geany. If Geany were to add ST3's indexing and fuzzy searching I'd switch back in a heartbeat.

[1]: https://github.com/jnvsor/TabNukerNuker
