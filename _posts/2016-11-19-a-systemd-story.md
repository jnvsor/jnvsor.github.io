I wrote an article suggesting that we would look back at Systemd in 30 years the way we look back at X11 today.

I apologize.

It's probably going to be earlier than that.

## What prompted this?

A phone company stated they'd be [switching from Debian to Devuan](https://neo900.org/news/2016-week-47):

> Maemo pre-dates the mandatory use of systemd in Debian and has relevant core functionality that depend on cgroups, a Linux kernel functionality (which is not 100% available under systemd seizing cgroups.)

The short history behind this is that Systemd co-opted cgroups to use them in a hacky way they weren't designed for, and it's breaking things. If you've heard the word "Systemd" before you've probably heard it in a similar context.

[Someone commented that](https://www.reddit.com/r/linux/comments/5drq5j/neo900_news_2016_week_47_whats_coming_to_the/da6zkqa/):

> nothing in Debian is stopping you doing `apt-get remove systemd && apt-get install sysvinit`

My immediate thought was of course: Yeah right. This is bullshit. It's an init system not a board game.

Doubting this statement I was immediately beset by multiple Systemd fanboys saying "You have no proof!" And "You're spreading anti-systemd hate!" as if I'm some sort of neo-nazi for pointing out when someone's objectively full of shit.

I didn't have to defend myself though, since someone else was brave enough to try the command and post the big fat error message right after me.

## What's wrong with systemd?

A better question is what's right with Systemd?

It's frequently compared to sysvinit because if it were compared to the actual competitors around the time of its inclusion in Debian (OpenRC, runit, Upstart) it would bring nothing to the table but NIH.

We know for a fact it isn't faster because its competitors boot just as quickly while using the same shell scripts Systemd supporters claim is causing a slow boot.

I would go so far as to suggest the only reason it has the following it has today is because of the same kind of people that replied to me - fervent fanboys manipulating (Or creating the illusion of) public support.

The inclusion of Systemd in Debian somewhat reminds me of how Debian aided the failed hostile takeover of FFmpeg and had to move everything back afterwards.

As I stressed tongue-in-cheek in my [previous article](/blog/2016/09/08/a-brief-history-of-systemd/), we are guaranteed to look back at Systemd the way we look back at X11 today.

We'll see its bloat and its scope creep and we'll spend a decade pulling it apart piece by piece to undo the damage it's caused, just like we've done with X11 for the last decade.

But there are arguments against Systemd today.

## The basic arguments

* Systemd is flaky. Brought to you by the guy behind Pulseaudio - that wonderful ball of instability we all know and love.
* Systemd developers don't care about breaking things, and tell everyone else to change their code to fit Systemd. You can google any number of things Systemd has unilaterally broken. Linus has [personally blasted them](https://lkml.org/lkml/2014/4/2/420) for this behavior and the response from Lennart was to claim that [Linus is toxic](https://plus.google.com/app/basic/stream/z13rdjryqyn1xlt3522sxpugoz3gujbhh04) because of the righteous fury he bestows on people who break the Linux kernel. And userland. Again. Like Systemd devs.
* Systemd has ridiculous scope creep. While we laugh at X11 for including a print server, Systemd now has user authentication, networking, mounting, containerization, hardware initialization, logging, and *oh yeah there's an init system tacked on to it too!* And it's getting more. It should outstrip X11's peak scope some time in 2019.
* Systemd runs as PID1.

## PID 1

Systemd is PID 1.

Bringing up the fact that authentication, networking, and PID 1 are all the same thing is the ultimate nightmare for anyone who knows what those things are and cares remotely about security and/or stability.

It's also the trigger for thousands of replies pointing out that "Only a small part of Systemd is PID 1!"

And this is true. Only a small part of Systemd is in PID 1. But it is coupled to all the other parts, so if one of them becomes compromised you have a very real security problem.

And yes, they are all coupled.

Whenever complaints arise about the scope creep of yet another part of the Linux ecosystem that's absorbed by Systemd, its apologists immediately point out that there are good reasons for doing this. These good reasons imply coupling to PID 1, or to a Systemd service that is itself coupled to PID 1. That's the only good reason there is to be part of Systemd.

As practical demonstration of how this is a real problem, Andrew Ayer pointed out you can [hang PID 1 with a normal user command shorter than a single tweet](https://www.agwa.name/blog/post/how_to_crash_systemd_in_one_tweet) by calling a notification utility.

Why does Systemd need to be in PID 1 in the first place? As Rich Felker [aptly put it](http://ewontfix.com/14/) before publishing a 22 line init:

> Do away with everything special about pid 1 by making pid 1 do nothing but start the real init script and then just reap zombies

The creator of competing init program runit [said](https://www.reddit.com/r/linux/comments/54yfcd/how_to_crash_systemd_in_one_tweet/d861zpe/):

> There is *one* compelling reason for this and that is that you can now supervise services that perform a double fork to background themselves.

So the only reason for PID 1 is to supervise services that perform a double fork. But we'll come back to that later...

## Cgroups

So why does Systemd even *use* cgroups? To quote [Poettering's original Systemd announcement](http://0pointer.de/blog/projects/systemd.html):

> Traditionally on Unix a process that does double-forking can escape the supervision of its parent, and the old parent will not learn about the relation of the new process to the one it actually started.
>
> [...]
>
> So, how can we keep track of processes, so that they cannot escape the babysitter, and that we can control them as one unit even if they fork a gazillion times?
>
> [...]
>
> Well, since quite a while the kernel knows Control Groups (aka "cgroups").
>
> [...]
>
> If a process belonging to a specific cgroup fork()s, its child will become a member of the same group. Unless it is privileged and has access to the cgroup file system it cannot escape its group. Originally, cgroups have been introduced into the kernel for the purpose of containers
>
> [...]
>
> It should be noted that systemd uses many Linux-specific features, and does not limit itself to POSIX. That unlocks a lot of functionality a system that is designed for portability to other operating systems cannot provide.

So long story short, Systemd uses cgroups as a hack to make sure daemons die when they're told. Except like a lot of Systemd's fundamental concepts: It's dead wrong.

Remember that old quote?

> About that systemd puts process supervision in pid1. There is *one* compelling reason for this and that is that you can now supervise services that perform a double fork to background themselves.

To quote the guy that responded to him:

> you can use [proc events](http://bewareofgeek.livejournal.com/2945.html) to get that information, since [2005](https://lwn.net/Articles/157150/)

5 years before Systemd was announced, the proper way to do it was added to Linux.

They sacrificed portability to use "many Linux-specific features" and didn't notice the Linux-specific feature that makes the entire reason for Systemd being PID 1 irrelevant.

Ladies and Gentlemen. I give you: Systemd!



PS: Go read more of [the runit guy's comments](https://www.reddit.com/user/goedkope_teringslet?sort=top), they give an interesting (And balanced) look at the technical details behind what Systemd does right and wrong
