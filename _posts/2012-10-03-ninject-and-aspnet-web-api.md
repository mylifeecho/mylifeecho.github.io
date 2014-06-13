---
layout: post
title: "Ninject and ASP.NET Web API"
description: "WebAPI was appeared in beta version of ASP.NET MVC4 with a new namespace. System.Web.Http is an analogue of System.Web.Mvc namespace with ApiController class, action filters, and many other similar classes with which we are familiar from the ASP.NET MVC. There is IDependencyResolver that is an analogue of IDependencyResolver from System.Web.Mvc. And I find it necessary to write my own realization of IDependencyResolver for Ninject."
category: "notes-tips-tricks"
tags: ["ASP.NET WebAPI", "Ninject"]
---
{% include JB/setup %}

WebAPI was appeared in beta version of ASP.NET MVC4 with a new namespace. **System.Web.Http** is an analogue of **System.Web.Mvc** namespace with ApiController class, action filters, and many other similar classes with which we are familiar from the ASP.NET MVC. There is **IDependencyResolver** that is an analogue of **IDependencyResolver** from System.Web.Mvc. And I find it necessary to write my own realization of IDependencyResolver for Ninject.

Or find it! (on GitHub: [example][1])

[1]: https://github.com/filipw/Ninject-resolver-for-ASP.NET-Web-API