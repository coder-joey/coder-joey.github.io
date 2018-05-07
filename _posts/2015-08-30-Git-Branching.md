---
layout: post
title: Git Branching
---

When working on a project it’s handy to leave the master branch in your repository as your “working copy”. Whenever a change needs to be made you can create a branch, mess about with the code in any way necessary, thoroughly test it and then merge that code into the master branch when you’re happy it all works.

New branches should have a meaningful name and in the console it’s as simple as:
```
$ git checkout -b branchname
```

The “-b” is short for “branch” and allows us to “checkout” and create this new branch all in one line.

Commits can be made on this branch as per normal but all changes will not affect the master branch. If you need to switch between branches you can use the “checkout” command:
```
$ git checkout master
$ git checkout branchname
```

This allows work to be carried out in multiple branches simultaneously without any conflicts. Conflicts can arise when merging back into master if developers have been working on the same code but these are dealt with when then. Until then, nothing in the master branch will be changed.

When you are ready to merge your branch into master (making sure everything is working as expected) simply run:
```
$ git checkout master
$ git merge hotfix
```

Once any conflicts have been resolved and the master is all up to date you can tidy your repository by deleting the merged branch:
```
$ git branch -d hotfix
```

More information [here](https://git-scm.com/book/en/v2/Git-Branching-Basic-Branching-and-Merging) .