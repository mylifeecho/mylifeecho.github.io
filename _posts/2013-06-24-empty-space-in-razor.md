---
layout: post
title: "Empty space in Razor"
description: ""
category: "notes-tips-tricks"
tags: ASP.NET MVC, Razor
---
{% include JB/setup %}


When we use Razor view engine to generate text trying to keep well formated code like this:

```html
<div>@foreach(int i in Enumerable.Range(1, 4)) {
	@i
	if(i % 2 == 0) {
    	<text> is even</text>
	} else {
    	<text> is odd</text>
	}
}</div>
```

we often get something like this:

```html
<div>1         is odd;
2         is even;
3         is odd;
4         is even;
</div>
```

To fix it you can use one more curly bracket to open section and one closing bracket to remove unnecessary symbols:

```html
<div>@foreach(int i in Enumerable.Range(1, 4)) {
   	@i
   	if(i % 2 == 0) \{\{
       	}<text> is even;</text> {
   	}} else \{\{
       	}<text> is odd;</text> {
   	}}
}</div>
```

and get this: 

```html
	<div>1 is odd; 2 is even; 3 is odd; 4 is even; </div>
```

Mustached Razor! =)
