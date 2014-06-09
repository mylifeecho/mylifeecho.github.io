---
layout: post
title: "Month names in .NET"
category: "notes-tips-tricks"
tags: [".NET"]
description: To get text name of a month from its ordinal number, you can easily use
---
{% include JB/setup %}

To get text name of a month from its ordinal number, you can easily use:

```csharp
CultureInfo.CurrentCulture.DateTimeFormat
	.GetMonthName(DateTime.Now.Month);
```

As it is presented in [MSDN][1], you can get the name of months, days and eras, by the means of the class **DateTimeFormat**. However, I can not say properly about how it works with eras...

[1]: http://msdn.microsoft.com/en-us/library/6thdx8xz