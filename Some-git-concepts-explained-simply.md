Some git concepts explained simply
==================================

This just a brief tutorial to explain simply some of the important features of
git that may be hard to grasp for newbies. They were hard to grasp for me.
This assumes familiarity with SVN. I write this to explain things briefly,
from my experience, not from the perspective of someone who knows nearly
everything about git, because the git related readings I found do not explain
things clearly and git is quite complicated. This reading is not to be taken
as definitive, it is guide from a mediocre git user and I use git and SVN
terms like "revision" and "commit" interchangeably. What is called "revision"
in SVN is called "commit" in git. But there is also the action of committing,
which is somehow different from the act of committing in SVN.

If you are new to git I suggest you start with a good GUI client, like SourceTree,
and take the concepts one by one as you move along.

SSH keys
--------
There are different ways to authenticate with git, but the most reliable one
is using SSH keys. There is help else where so I just want to explain the
idea. You generate a key, it has private and public part. The private part
stays with you and you add the public part to your GitHub account (or other
git hosting provider). This way your account becomes associated with the key
and whenever you use git, it will authenticate with the key and will have the
same rights to the repositories as your user account. For more info
[check here](https://help.github.com/articles/generating-ssh-keys).


Local repositories
------------------
With SVN things are simple - there is SVN repository on the server and there
is working copy locally. We work and we commit the work to the repository.
With git we have at least two repositories. One remote (the server) and one
local where we work. When we commit, we commit to our local repository - it
has revision history and everything, just like the server, it is like we run a
local git server. And so it is not in any way connected to the remote one -
besides that we can setup the remote repository somewhere in our config for
syncing. So with git we have additional step called push. Push is like commit
in SVN. Push pushes the changes we committed to our local repository to the
remote one. If we didn't commit locally we have nothing to push. And if we
commit but don't push our changes are not on the server.

We can use this, for example, to split our work into multiple commits. Let say
we have 3 small tasks to finish. We finish one task and we commit locally, so
we have this revision in history - we have the changes related to this task in
a separate commit with separate commit message. Then we do another task and do
the same. Only when we finish what we want for the day, e.g. our 3 tasks, we
push the changes to the remote repository. Before we push nobody else can see
our changes even though we did commit. And when we push we push 3 commits
(revisions) to the remote, not only one.


Staging area
------------
Besides that git has local repository it has staging area. I'm not going to
explain this because I don't use the staging area myself. It is optional. It
is like an extra step before our local changes are committed. For me this step
is redundant because it is like second local repository, but maybe it is just
that I haven't realized how to use it properly. In any case I mention this
because this article is aimed for newbies and the staging area is something
that confused me. So I just want to say this tip - you can disable the staging
area if you don't need it or don't properly understand it (in the case of
SourceTree which I use, it has to be "disabled", it could be other term in
other client or CLI git).


Sub modules
-----------
This is my favourite git feature and why git won me over SVN. It is not
without downsides but it is great way to reuse projects. Sub modules are
repository inside the repository - but it is not like we just copied the files
manually, but rather a link to specific version of other repository. I can
have one project and put it as sub module in another project. Then when I work
on the second project when I make changes in the files that belong to the sub
module I can commit these files directly to the original project. Actually
there is no other way, if I change files in the sub module I can't commit them
to the parent module, because the sub module is just a link, not a copy.

To illustrate this otherwise it like we have two local repositories in
separate directories and we need to reuse some files. So we create a symlink
in one repository to the other to make all changes in the same place and avoid
having to copy files back and forth. Only that this is integrated into git so
it is more useful.

This can also lead to some confusion. Because sub modules are link to another
repository - this is easy - but they also have version. So when I change
something in the sub module, I commit it to the original repository, but the
version in the parent repository is not changed. So I need to commit two times -
once in the sub module to say "make these changes in the sub module" and
once in the parent repository to say "make this sub module point to the newer
version that has my latest changes". If this extra step wasn't necessary
someone else can change the sub module and I will be forced to use the the
latest revision always. The extra work of having to do two commits can be
annoying and one can forget, particularly when working with many sub modules,
but the extra flexibility is worth the cost. Also if you use the right tools,
e.g. SourceTree, your git client will remind you and help you commit everything
properly.


Rebase
------
Rebase helps to have cleaner commit (revision) history. With SVN, usually to avoid
conflicts, one does an update before starting to work and makes his/her
changes on top of the latest revision. But even then it is possible that while
we work on one file someone else modifies the same file in the SVN repository.
With SVN we have to merge the changes somehow (maybe after we resolve some
conflicts) and we can commit our own changes then. But git is different
because we commit our changes to the local repository which is not necessarily
up-to-date with the remote repository. So in the same scenario - we do some
changes on top of the latest revision and while we are doing them someone else
pushes a new revision. We can still commit our changes locally because we have
the older version. But then we have the same problem like with SVN when we try
to push (to synchronize our local changes with the remote repository). So
again we need to merge the changes somehow. How do we do this? We need to pull
the remote changes, but when we pull on top of the changes we already did we
are creating a second commit, i.e. revision, locally. So we did one change but
we have two revisions. One of these revisions is redundant because it only
servers to merge someone else's changes. The result is unclean history full of
default commit messages like "Merge remote branch master", which hold no
value.

Rebase solves this problem by re-writing commit history in our local
repository. Instead of creating a new commit on top of our newest changes, it
pull the changes from the remote repository and writes them before our own. In
other words it is like we started working after the latest changes and nobody
else did any changes before we commit. In yet another words, it arranges the
commits in the order they happened and we have a cleaner history without the
extra commit.

Rebase must be used with caution, at least my GUI client says so, and this is
not meant to explain everything about rebase, but just to try to give some
insight into the idea behind it.


More
----
There is much more to learn about git. There are areas of git that I never
even touched myself, and even after I used it for over an year I keep hearing
about some feature that I wasn't aware of. I just wanted to share some of my
experience that has worked well for me in hope it can save some woes to
others.



Authors
-------
Borislav Peev (borislav.asdf at gmail dot com)