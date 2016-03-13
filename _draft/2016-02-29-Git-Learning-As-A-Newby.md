---
layout: post
title: Learning Git as a newbie
Category: Git
Tag: Source Control, Git
---

I used a lot of source control in the past, from Visual Source Safe, SVM, clearcase or perforce. They all have their pro and cons and scripting and automating task helped as well to approche it from a deeper view. The open source community and poeple today are slowy moving to [Git][https://git-scm.com/] source control. One of the well know hosting platform is [Github][https://www.github.com] which I'm using to host this blog.
As a newby in Git source control I want to share what the finding in the way of working as well as some of the basic of Git. Everything I say may not be completly accurate, however this is what I found so far. I know they are a lot of poeple who posted about Git and it's usage, and I just want to have my own note about it.

# The Basic
Git is a source control that doesn't need you to have access to the central repository to work and commit code. All the action are everytime happening on your local copy of the repository and are not pushed until you are doing it. On top of that every local copy (named clone) contain all the history about the repository. This mean that you can check the history of files, commit, revert and and so on.

## Commands

### Git Clone

