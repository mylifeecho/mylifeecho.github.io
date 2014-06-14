---
layout: post
title: "Life cycle management in Ninject"
description: "Configuration of object's life-time in Ninject is set by invocation of appropriate method as it's shown below"
category: "notes-tips-tricks"
tags: ["IoC", "Ninject"]
---
{% include JB/setup %}

Configuration of object's life-time in Ninject is set by invocation of appropriate method as it's shown below:

```csharp
kernel.Bind<IFileSystem>().To<FileSystem>()
    .InSingletonScope();
```

List of predefined methods:

1. `InScope` - delegate Func, which defines life time of an object, is passed to this method. Until the object returned by delegate is not disposed and the same one, the same instance of the object will be used. It is most common method and the other methods use it.
2. `InTransientScope` - new instance is created for each request. It is the same as `InScope(c => null)`.
3. `InThreadScope` - new instance is created per each thread. It is the same as `InScope(c => System.Threading.Thread.CurrentThread)`
4. `InSingletonScope` - the name speaks for itself. It is the same as `InScope(c=> c.Kernel)`
5. `InRequestScope` - new instance is created for each HttpContext. The same as `InScope(c => System.Web.HttpContext.Current)`
