---
layout: post
title: Useful Git commands (Part 2)
date: 2018-01-20 10:00 +0530
---
## GIT SQUASHING
There are instances when a feature development is in progress in the staging branch, there exist multiple commits in the staging branch.
In the below example, three different commits are performed.
```
(karthi)[git-example]$ touch a.txt
(karthi)[git-example]$ git add a.txt
(karthi)[git-example]$ git commit -m "commit A"
[master (root-commit) 24b59b0] commit A
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 a.txt
(karthi)[git-example]$
(karthi)[git-example]$ touch b.txt
(karthi)[git-example]$ git add b.txt
(karthi)[git-example]$ git commit -m "commit B"
[master deee2fa] commit B
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 b.txt
(karthi)[git-example]$
(karthi)[git-example]$ touch c.txt
(karthi)[git-example]$ git add c.txt
(karthi)[git-example]$ git commit -m "commit C"
[master 7fa9efd] commit C
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 c.txt
```
Lets check the commits using "git log"

```
(karthi)[git-example]$ git log -l
fatal: Option 'l' requires a value
(karthi)[git-example]$ git log
commit 7fa9efd1c6dfe65b733ad32293786cd0bd020534
Author: Karthikeyan Arulmozhivarman <karkey85@gmail.com>
Date:   Fri Jan 19 09:17:51 2018 -0500

    commit C

commit deee2fabe37113224bf4926481b691be4bc098ad
Author: Karthikeyan Arulmozhivarman <karkey85@gmail.com>
Date:   Fri Jan 19 09:17:51 2018 -0500

    commit B

commit 24b59b0ed26d03fa498569e3bb03d7c7b9e4ffe1
Author: Karthikeyan Arulmozhivarman <karkey85@gmail.com>
Date:   Fri Jan 19 09:17:51 2018 -0500

    commit A
```
In this case, we have an option in git to squash these multiple commits. First, make sure which commit is pointing to HEAD.

```
(karthi)[git-example]$ git show HEAD
commit 7fa9efd1c6dfe65b733ad32293786cd0bd020534
Author: Karthikeyan Arulmozhivarman <karkey85@gmail.com>
Date:   Fri Jan 19 09:17:51 2018 -0500

    commit C

diff --git a/c.txt b/c.txt
new file mode 100644
index 0000000..e69de29
```

Lets perform squash the first 2 commits "commit B" and "commit C" into a single commit.
```
(karthi)[git-example]$ git rebase -i HEAD~2
pick deee2fa commit B
pick 7fa9efd commit C

# Rebase 24b59b0..7fa9efd onto 24b59b0
#
# Commands:
#  p, pick = use commit
#  r, reword = use commit, but edit the commit message
#  e, edit = use commit, but stop for amending
#  s, squash = use commit, but meld into previous commit
#  f, fixup = like "squash", but discard this commit's log message
#  x, exec = run command (the rest of the line) using shell
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#
# Note that empty commits are commented out
```

Now change the second commit keyword pick to squash and save/close the editing session.

```
pick deee2fa commit B
squash 7fa9efd commit C
```

Now another editing session opens where a new commit message can be given.

```
# This is a combination of 2 commits.
# The first commit's message is:

commit B

# This is the 2nd commit message:

commit C

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
# HEAD detached at deee2fa
# You are currently editing a commit while rebasing branch 'master' on '24b59b0'.
#
# Changes to be committed:
#   (use "git reset HEAD^1 <file>..." to unstage)
#
#       new file:   b.txt
#       new file:   c.txt
#
```
The new commit message "commit BC" is given as below and save/close the editing session:

```
# This is a combination of 2 commits.
# The first commit's message is:

commit BC

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
# HEAD detached at deee2fa
# You are currently editing a commit while rebasing branch 'master' on '24b59b0'.
#
# Changes to be committed:
#   (use "git reset HEAD^1 <file>..." to unstage)
#
#       new file:   b.txt
#       new file:   c.txt
#
```

Now the commits are squashed to a single commit. Do "git log" as shown below:

```
(karthi)[git-example]$ git rebase -i HEAD~2
[detached HEAD 35efe8d] commit BC
 2 files changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 b.txt
 create mode 100644 c.txt
Successfully rebased and updated refs/heads/master.
(karthi)[git-example]$
(karthi)[git-example]$ git log
commit 35efe8d72976966e5a317cfa829a603fafa55c12
Author: Karthikeyan Arulmozhivarman <karkey85@gmail.com>
Date:   Fri Jan 19 09:17:51 2018 -0500

    commit BC

commit 24b59b0ed26d03fa498569e3bb03d7c7b9e4ffe1
Author: Karthikeyan Arulmozhivarman <karkey85@gmail.com>
Date:   Fri Jan 19 09:17:51 2018 -0500

    commit A
```

Final word. we may need to perform force git push in these cases for git push to succeed. 

```
git push -f
```

Happy squashing!!!
