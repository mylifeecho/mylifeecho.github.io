---
layout: post
title: "RazorEngine"
description: "You can use *.cshtml templates to create text content, for example for massive email sending slightly different for each user. There are at least two ways to do it."
category: "notes-tips-tricks"
tags: ["ASP.NET MVC", "Razor"]
---
{% include JB/setup %}

You can use **.cshtml* templates to create text content, for example for massive email sending slightly different for each user. There are at least two ways to do it.

1. You can make it something like this:

```csthml
public static string RenderPartialViewToString(Controller controller, string viewName, object model) {
    controller.ViewData.Model = model;
    using (StringWriter sw = new StringWriter()) {
        var viewResult = ViewEngines.Engines.FindPartialView(controller.ControllerContext, viewName);
        var viewContext = new ViewContext(controller.ControllerContext,
                                          viewResult.View,
                                          controller.ViewData,
                                          controller.TempData,
                                          sw);
       viewResult.View.Render(viewContext, sw);

        return sw.GetStringBuilder().ToString();
    }
}
```

But we need a controller context in this case. And as an advantage of this method is correct work of all Html and Url helpers in template. Of course, if we use context of real controller but not dummy.

2. Also we can use [RazorEngine][1], but without using helpers. If you are going to use them, you should know that there is a tricky way to interact with helpers([link][2]).

```cshtml
string template = "Hello @Model.Name! Welcome to Razor!";
string result = Razor.Parse(template, new { Name = "World" });
```

You may download it from NuGet. Just type it in to your package manager console: **Install-Package RazorEngine**.

[1]: http://razorengine.codeplex.com/
[2]: https://github.com/Antaris/RazorEngine/issues/29