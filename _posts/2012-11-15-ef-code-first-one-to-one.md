---
layout: post
title: "EF Code First: One to One"
description: ""
category: 
tags: []
---
{% include JB/setup %}

Entity Framework Code First has a couple of options to create One-to-One relationship between objects. The one of them when a primary key of one object is a primary key and foreign key of another object at the same time.

```csharp
public class Page {
    [Key]
    public Guid Id { get; set; }
    public Content Content { get; set; }
    
    public virtual PageRevision Revision { get; set; }
}

public class UserRevision : BaseRevisionEntity {
    [Key, ForeignKey("Page")]
    public Guid PageID { get; set; }

    public virtual Page Page { get; set; }
}
```

Also you have an ability to create one-to-one relations, but stracture of database will be similar to the structure of classes with inheritance. If we want to represent inheritance in database, we can use the attribute [Table] for all children with name of tables. Code below describes it: 

```csharp
public class BaseEntity {
    public Guid Id { get; set; }
    public bool IsDeleted { get; set; }
}

[Table("User")]
public class User : BaseEntity {
    public string Name { get; set; }
}

[Table("SomeItem")]
public class SomeItem : BaseEntity {
    public string Content { get; set; }
}
```

If you do not use `[Table]` attribute for each entities, EF will create a table with full list of properties combined from all classes in inheritance chain.

[This good article][1] is about relations between object and how to configure it in Entity Framework Code First

[1]: http://weblogs.asp.net/manavi/archive/2011/03/27/associations-in-ef-4-1-code-first-part-1-introduction-and-basic-concepts.aspx