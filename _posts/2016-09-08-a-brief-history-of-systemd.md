---
displaydate: 2045-04-25
---

It's now 2045 - 30 years since the widespread adoption of systemd.

When it was released, systemd met no small resistance due to it's disregard for the underlying ecosystem's "Unix philosophy" in that it dropped an awful lot of functionality into a large complex process.

This was considered by all at the time to be a bad thing, but the pro-systemd advocates loudly proclaimed that the features outweighed the poor code quality and highly coupled architecture.

## The honeymoon

A few years after launch systemd was in ubiquitous use, and the vast majority of users loved it. Features were added to the cautious optimism of the former naysayers and a large portion of programs in use on Linux systems came to depend on it.

The features which no-one else had managed to match were a great leap forward, and eventually the distrust of systemd died down over the years, occasionally resurfacing after a large-scale security issue or system-breaking bug, but overall attitude towards systemd was ubiquitously favorable. At first.

## The bloat

Entering it's 8th year as the default init system on Debian systems, systemd continued to accrue features. These features were occasionally of questionable usefulness, including a built-in network stack for remote system control, and even a print server (Yes, they still used printers back then!)

Systemd maintenance had become a nightmare. It was far easier to pile on bad code than to prune the code that was already there, and some direly needed features were impossible to add to the architecture.

While the system did see features added over the course of years, these were delicately balanced hacks which - like all hacks - should not have been in use for as long as they were.

Systemd began to acquire infamy beyond even it's pre-adoption era. *The Linux-Haters Handbook* (2028) devoted a full chapter to the problems of systemd. *Why Systemd is not our ideal init system* (2024) detailed problems in the system with recommendations for improvement.

It's popularity nowhere to be found and it's flaws laid bare, the time was ripe to start from scratch. Unfortunately, it took more than a decade.

## The light at the end of the tunnel

Systemd Foundation members began work on a new init system with a more decoupled init system in 2034, with the 1.0 release under the name *Sayland init system* 3 years ago. A vastly simplified init system, it's architecture was one no-one could have imagined at the time of Systemd's inception.

While there was a brief attempt to hijack the progress in Sayland by Canonical Corporation's Minit, the majority of Linux distros switched to Sayland quickly, with Red Hat-backed Beret adopting the Sayland init system last year. Debian is expected to ship it by default in Debian 22 Zorg.

## Repeating past mistakes

We should be careful not to repeat past mistakes. While the SynuxD kernel project is ambitious and provides many benefits over the ubiquitous and antiquated Linux kernel, I can't help but feel the focus on features will be a similar detriment to Security, Stability, and long-term progress as Systemd turned out to be.

I am not very hopeful on this front though - if software developers were such good studies of history they might have avoided the Systemd disaster with a brief glance at the X11 project, which had many surprising similarities to Systemd, and was also scrapped after 3 decades of bloat right around the time of Systemd's rise to power.
