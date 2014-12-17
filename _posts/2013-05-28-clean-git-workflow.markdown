---
layout: post
title:  "Clean is not only for code or test"
date:   2013-05-28 17:00:00
categories: git workflow
---

**TL; DR**
**Branching -> Cleaning-up commits -> Merging -> Pulling -> Pushing;**

Working in a clean and neat environment is not ONLY for the production code and the tests of the software, it also can apply on different aspects of software development process. One of these many different aspects can be Version Control that you’re using! How clean is your history/branches/check-ins/commits/etc.? It’s very important to make the environment that we’re working in SUPER clean. Cause it makes the development process much smoother and faster and more efficient. Always remember that the only way to go fast, is to go clean! And clean is not only for code!

I use Git for version control most of the times (I believe at this moment, EVERYONE is using some sort of version control software and if someone doesn’t, then God help them). And after different kinds of experiments and ways of working with it (both as a solo developer and as a team member) I found a clean, neat, smooth and headache-less (that’s not even a word) workflow with Git. I’m going to share it with you here so probably someone finds it useful. There are hundreds of articles and blogs on how to work with Git in a better way and this is only my personal preferred way of working which happens to work very well for me and I’m sure it’s going to be the same for someone else.

Here’s how it works.

Imagine we're working on a software program and we have 2 remote branches for it! One is the "master" (which we want it to be as clean as possible) and another is "foo_feature" which we branched it out of master in order to implement feature "foo" (whatever that is)!

Obviously we have the corresponding local branches of these two remote ones on our own machine and we work on them locally whichever branch that we’re on at any moment. Now if I want to work on something related to “foo feature”, this is my workflow for doing it:

First I pull the latest changes from the remote foo-feature branch to my local foo-feature branch:

**git pull origin –rebase foo-feature**

I’m pretty sure a lot of feature are against using –rebase for a lot of reasons which I’m not gonna mention here but the reason that I like using it most of the times is that it’ll give me the ability to solve merge conflict issues one step at a time and continue the process and at the end I won’t have that one extra merge commit which is being generated automatically by Git when you pull. Also it put my latest changes on top of the latest changes in the remote branch history.

Anyway, then I create a new local branch on my machine out of the latest version of foo-feature (I rather not mess with the local foo-feature repository during my experiments and keep it clean and do! So here's how’s it’s gonna work: (imagine I want to do some Refactoring on the code)

**git checkout -b foo-feature-refactoring**

After that, I start making my changes and commit them as much as I need during this Refactoring and when I get a log from my foo-feature-refactoring repository imagine here's what I get as a result:

**git log –oneline**

**e329shf commit message 1**

**e329shf commit message 2**

**e329shf commit message 3**

**e329shf commit message 4**

**…**

At this point I'm done with the Refactoring and I want to merge it back to foo-feature branch and push it to the remote branch named foo-feature so other people in the team can see them as well! But since I was experimenting a lot during this Refactoring and made some mistakes I ended up with some commits that I don't want to be in the clean history of foo-feature! I’m sure this happens to everyone during the development. (Some of these commits are not providing any value by being in the history and they’re just bunch of noise there)

So I try to make all these commits into few nice and meaningful commits which totally make sense and they provide some value if they'll be in foo-feature repository. Beautiful Git let me to do it like the following:

**git rebase -i HEAD~4**

This will give me the last 4 commits that I made! It will open up an interactive environment (editor) for me including my last 4 commits (which I showed above as a result of git log --oneline) and I see something like the following:

**pick e329shf commit message 1**

**pick e329shf commit message 2**

**pick e329shf commit message 3**

**pick e329shf commit message 4**

**# Commands:**

**# p, pick = …**

**# r, reword = …**

**# …**

**# s, squash = use commit, but meld into previous commit**

**# …**

As you can see it gives the list of commit messages in an ordered manner (the oldest at the top) and a set of commands that we can manipulate the commits with them!
The one that I'm interested in here for this use case is "s, squash" which will meld the commit into the previous one! Now I Refactored the code in my local branch named foo-feature-refactoring and I want to have only 1 commit with a clear message about what I did so I'll edit the text that git generated for me to the following:

**pick e329shf commit message 1**

**squash e329shf commit message 2**

**squash e329shf commit message 3**

**squash e329shf commit message 4**

**# …**

After doing that all these 4 commits will be meld into one commit and git will give me another editor withe the following structure so I can write my commit message for this whole action:

**# This is combination of 4 commits**

**# The first commit's message is:**

**commit message 1**

**# The first commit's message is:**

**commit message 2**

**# The first commit's message is:**

**commit message 3**

**# The first commit's message is:**

**commit message 4**

**--> Refactored the FooFeature, got rid of some duplication (DRY).**

**# Please enter the commit message for your changes. …**

**# …**

Now after save/quit of this editor I have only one commit with the message "Refactored the FooFeature class, got rid of some duplication (DRY)." which is succinct and clear. I got rid of all those noisy commits (message 1, message 2, etc.)

Now I need to merge this thing back to my local foo-feature branch which is a 2 step process:

**git checkout foo-feature # switched to foo_feature branch**

**git merge foo-feature-refactoring # merged those 2 branches**

Now I can delete the foo-feature-refactoring or if I need it, I'll keep it there in the collection of local branches! (depends on the scenario obviously)

Now I need to push this change to the remote branch origin/foo-feature so others can see what's going on! I just do a pull first (after I committed anything I have in my working tree of course):

**git pull –rebase origin/foo-feature #and solve any conflict if any exists**

Then I push my local changes to the remote branch:

**git push origin/foo-feature**

Now if someone else pulls the origin/foo-feature they will see my commit:

**e329shf Refactored the FooFeature class, got rid of some duplication (DRY).**

Instead of 4 commits with confusing messages which are just messing the history and log of the repository!

Also for having better messages in the commits I highly recommend reading these great points by Tim Pope on his blog [here](http://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html)!

Maybe a lot of you are already working in this way! I just wanted to share this approach with you since I found it super useful and clean during the development process SPECIALLY in a team.

Worth a try! Hope that helps.

- Special thanks to Phil Corliss for telling me some interesting points about having a nice workflow in Git and its benefits over time!

- If you haven't already, definitely read [this](http://git-scm.com/book/en/Git-Branching-Rebasing) part of "git" book/documentation on Rebasing and why it's powerful & when you should not use it!
