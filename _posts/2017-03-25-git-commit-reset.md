So occasionally you might be working with people who have a less-than-stellar grasp of version control, who merge an old branch with `--ours` or worse - copy paste their entire repo deleting all the work of the last month.

Was that a bit too specific? Anyway, when this happens you might run into a group of commits you need to revert.

Ordinarily this is just `git revert fecced` but if there are multiple merge commits and normal commits mixed in with all sorts of nightmares you might want a reset. Unfortunately, a reset implies a force push which is a bad thing&#8482; for many reasons.

If only there was a way to make a revert commit.

## Introducing! <small>git commit-reset</small>

<script src="https://gist.github.com/jnvsor/31596e0679cc6aa9701fc5541f6d6457.js"></script>

This is an alias that will make a commit that "Reverts" the current branch to a previous commit. It effectively undoes everything and sets your whole tree to that version.

Before:

{% highlight bash %}
$ git log --oneline --stat
e0c202d Fuck everything up, but worse
 file | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)
3251311 Fuck everything up
 file | 5 +----
 1 file changed, 1 insertion(+), 4 deletions(-)
0587e18 Change file
 file | 5 ++++-
 1 file changed, 4 insertions(+), 1 deletion(-)
6c33024 Add file
 file | 1 +
 1 file changed, 1 insertion(+)
$ git rev-parse HEAD^{tree}
b97b0c91c61e8fe3fc9a956797d12309d45710ec
$ git rev-parse HEAD^^{tree}
eb27a12891913c01d03907cbe6f9f8cdb7d65135
$ git rev-parse HEAD^^^{tree}
8be01bfae71f0b75c6dab647f283af4ede77ae06

{% endhighlight %}

Command:

{% highlight bash %}
$ git commit-reset 0587e18 Unfuck stuff
Updating e0c202d..dcdd38b
Fast-forward
 file | 5 ++++-
 1 file changed, 4 insertions(+), 1 deletion(-)
{% endhighlight %}

After:

{% highlight bash %}
$ git log --oneline --stat
dcdd38b Unfuck stuff
 file | 5 ++++-
 1 file changed, 4 insertions(+), 1 deletion(-)
e0c202d Fuck everything up, but worse
 file | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)
3251311 Fuck everything up
 file | 5 +----
 1 file changed, 1 insertion(+), 4 deletions(-)
0587e18 Change file
 file | 5 ++++-
 1 file changed, 4 insertions(+), 1 deletion(-)
6c33024 Add file
 file | 1 +
 1 file changed, 1 insertion(+)
$ git rev-parse HEAD^{tree}
8be01bfae71f0b75c6dab647f283af4ede77ae06
{% endhighlight %}
