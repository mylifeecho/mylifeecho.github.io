---
layout: post
title: "Fluent Builder in C#"
description: ""
category: dev
tags: [ C#, Design Pattern ]
---
{% include JB/setup %}

I like the pattern “Fluent builder”. This pattern provides an easy and transparent way to build complex objects with an hierarchical structure. If you are the developer of a builder, this pattern helps you to hide complexity of building objects, manage the subnode creation, define the order in which client should call methods of builder, where the order is important, and gives you other advantages of using “Builder” pattern, described in the book written by [GoF]({{BASE_PATH}}/books/#design-patterns). 

If you are the developer, who use a fluent builder at the first time, you have a better protection from the use of class in wrong way and your code will be more transparently. Also you have an easy way to investigate the interface of the builder on-the-fly (it is more convinient if you have something like IntelliSence).

Look at this 

```csharp
var doc = new SomeDocument();
doc.Initialize(param1, param2);

var subnode1 = new Subnode();
subnode1.SetType(type);
subnode1.SetName(name1);
subnode1.Bind(doc);

doc.AddNode(subnode1);

var subnode2 = new Subnode();
subnode2.SetName(name2);
subnode1.Bind(doc);

doc.AddNode(subnode2);
doc.Save();
```

And compare it with this:

```csharp
DocBuilder.CreateDocument(param1, param2)
    .AddNode(node => node
        .SetType(type)
        .SetName(name1))
    .AddNode(node => node
        .SetName(name2))
    .Save();
```

In the first case you may forget about the “Initialization” or “Bind” methods, forget to add a new node or add it correctly (we will see this case below in live example with [iTextSharp][itextsharp]), you have to define a lot of local variables and when you call method `Save()` you may try to add a new node and hope that it will be applied.

Now I’m going to describe how to implement fluent builder in C# to wrap a simple process of generation PDF document by using iTextSharp and get the code similar to that one presented above. Of course it may be more complex in your case and lead to a long invocation chain but this pattern should be used wisely as everything else.

First of all we should define what we are going to do. We will create PDF document with 2 pages which should have a image and empty background and small text messages. Thereby we have to create document and page builder classes and two interfaces of page builder to define the order of page building process. We will prohibit instantiation of page builder by client code with help of internal keyword before constructor to have more control over creation process and hide the complexity of this process.

```csharp
public class PDFBuilder : IDisposable
{
    private Document _document;

    public PDFBuilder(Stream stream)
    {
        _document = new Document();
        PdfWriter.GetInstance(_document, stream);
        _document.SetPageSize(PageSize.A5);
        _document.Open();
    }

    public PDFBuilder AddPage(Action<IFirstStep> buildPage)
    {
        // code to add page
    }

    public void Save()
    {
        _document.Close();
    }

    // IDisposable interface implementation
}

public interface IFirstStep
{
    ISecondStep BackgroundImage(string imagePath);
    ISecondStep LeaveEmptyBackground();
}
 
public interface ISecondStep
{
    ISecondStep AddParagraph(string text);
}
```

Creation of page with using [iTextSharp][itextsharp] is good example of the necessity to keep in mind order of calling methods to create page correctly. First of all we should set page size, then call NewPage method, set margins for all document to apply it for building page, and for a new page we should repeat all this actions. During creation of a page we don’t have any object of page, and this process is more procedural than object-oriented, but we hide it in our builder. 

You can see implementation of AddPage method below.

```csharp
public class PDFBuilder : IDisposable
{
    // ...
    
    public PDFBuilder AddPage(Action<IFirstStep> buildPage)
    {
        if (!_document.NewPage())
        {
            throw new InvalidOperationException("Unable to create page.");
        }
        _document.SetMargins(0, 0, 0, 0);

        buildPage(new PageBuilder(_document));

        return this;
    }

    // ...
}
```

The most important thing here is that argument of `AddPage` method is an action where we define how to build our page. Client code passes all information about the creation of page to the method of PDF builder which will be executed in due time. Notice that we pass to the buildPage action PageBuilder instance, but the type of defined in AddPage action is IFirstStep. This interface defines two opposit actions, so after one of them client code can add paragraphs to our PDF file. Listing below shows implementation of PageBuilder class.

```csharp
public class PageBuilder : IFirstStep, ISecondStep
{
    // ...
    public ISecondStep BackgroundImage(string imagePath)
    {
        var image = Image.GetInstance(imagePath);
      
        Rectangle pageSize = _document.PageSize;
        image.ScaleToFit(pageSize.Width, pageSize.Height);

        image.SetAbsolutePosition(0, pageSize.Height - image.ScaledHeight);
        _document.Add(image);
        return this;
    }

    public ISecondStep LeaveEmptyBackground() 
    { 
        return this; 
    }

    public ISecondStep AddParagraph(string text)
    {
        _document.Add(new Paragraph(text));
        return this;
    }
}
```

And how it looks for developer who uses this builder.

![example]({{ BASE_PATH}}/assets/posts/2013-05-27/flient-builder-use.png)

[itextsharp]: http://sourceforge.net/projects/itextsharp/