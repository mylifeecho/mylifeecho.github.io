---
layout: post
title: "GitHub + Bitbucket"
description: "GitHub and Bitbucket are two most popular hosting services for your projects. There are a few not very important differences, for example GitHub is more social than Bitbucket, but Bitbucket has support both Git and Mercurial. Both have an issue tracking, wiki, user-friendly interface, a good integration with different services. But there is one big difference and it is pricing plans."
category: dev
tags: [ GitHub, Bitbucket ]
---
{% include JB/setup %}

GitHub and Bitbucket are two most popular hosting services for your projects. There are a few not very important differences, for example GitHub is more social than Bitbucket, but Bitbucket has support both Git and Mercurial. Both have an issue tracking, wiki, user-friendly interface, a good integration with different services. But there is one big difference and it is pricing plans. 

If you are going to use GitHub you will pay for private repositories with unlimited team members and public repositories. Bitbucked plans based on number of collaborators with unlimited number of private and public repositories. If a company has a few main projects and appearance of a new project is not very frequent and unpredictable event, GitHub seems to be more convenient to use. But in outsourcing company projects appear one by one and Bitbucket is more suitable.

As for me (and I guess for any developer) I want to store all my projects somewhere, from small tests and investigation projects, prototypes, etc, to open source projects which are not ready to be published for some reasons. So I moved from GitHub to Bitbucket a few months ago and now there is no problem with a lot of repositories in my public profile that are interested only for me, but I still can use all advantages of private web-based DCVS hosting service for free. 

If you have recently started to use distributed version control system, you may not know that it is quite easy to move your open source project from Bitbucket to GitHub when the time has come. Just add new remote repository to your repository list, and push changes to this repository. A few clicks and you are already migrated from one service to another with full project history. Moreover you can push only special branch which will be used for releases and with help of rebase and squash features of git you can hide history of commits before pushing a new version to GitHub.

I hope I’ll do it in the future.