---
layout: post
title: "Octophoenix"
category: "dev"
tags: ["jekyll", "GitHub Pages"]
description: This blog was under reconstruction several months. Mostly because of a lot of things happened and I had no time to do all necessary reorganisation work related to the blog. Eventually I decided to reborn and migrate from Orchard CMS to Github Pages with Jekyll. Rise, my blog, in shape of Octophoenix! 
---
{% include JB/setup %}

This blog was under reconstruction several months. Mostly because of a lot of things happened and I had no time to do all necessary reorganization work related to the blog. Eventually I decided to reborn and migrate from [Orchard CMS][orchard] to [Github Pages][gp] with Jekyll. Rise, my blog, in shape of Octophoenix! 

<!--more-->
Orchard CMS is excellent opensource content management system from Microsoft. This CMS is good example how to build flexible and extensible software using ASP.NET MVC and C#, full of best practices, design patterns and interesting solutions. I learnt a lot investigating source code of Orchard CMS. This blog was based on Orchard CMS quite a long time. All posts until this one has been written using Orchard. But since I have been started I thought that I don't need all these features for blogging and I felt lack of such features as straightforward version control, possibility to keep all content in one place, but not like SQL for content and file system for pictures and other assets. Moreover any blog contains mostly static content and it's not necessary to have dynamic engine to serve your blog. Honestly to have acceptable response time using Orchard you have to have quite performant hosting, not cheapest one. Thus all this ideas lead to the conclusion to use Jekyll. 
Why to choose Jekyll?

1. You can use [GitHub Pages][gpj] to host your blog on Jekyll for free and it won't consume 5-15 USD per month (and buy subscription of cloud password manager for instance).

2. Whole blog is on file system with all text content, pictures, etc.

3. As result you can use git for version control (and you should for GitHub Pages).

4. You can use markup language to write posts instead of ugly way to use built in WYSWYG editor or write pure html.

Key notes:

1. I use [jekyllbootstrap][jb], but anyway I had to spent quite a lot of time to fix and customize everything for myself. There are several convenient rake tasks to create new posts, pages, drafts, built in analytic tool, comments, themes, etc.

2. I changed lifeecho theme to support of mobile devices and work with jekyll. This support is based on [Skeleton CSS framework][skeleton]. This framework contains only grid system, plus some minor features

3. For syntax highlighting I use Rouge. But keep in mind, you have to add styles manually. I have found CSS [here][css]

This blog post is first post created with using Jekyll, but hopefully not last one. 
One small thing, I have to migrate all data from Orchard based blog to new one...

[orchard]: http://www.orchardproject.net/
[gp]: https://help.github.com/articles/what-are-github-pages
[gpj]: https://help.github.com/articles/using-jekyll-with-pages
[jb]: http://jekyllbootstrap.com
[skeleton]: http://www.getskeleton.com/
[css]: http://richleland.github.io/pygments-css/
