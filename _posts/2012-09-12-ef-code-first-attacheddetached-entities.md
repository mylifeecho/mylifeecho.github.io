---
layout: post
title: "EF Code First Attached/Detached entities"
description: "POCO-classes in Entity Framework Code First might have two states: attached and detached. In the state attached to the context all changes are detected and in case they exist, the use of method SaveChanges() writes current state to database. If the object is not attached, the changes won't certainly be detected and saved to the database."
category: "notes-tips-tricks"
tags: [EF Code First]
---
{% include JB/setup %}

POCO-classes in Entity Framework Code First can have two states: attached and detached. In the state attached to the context all changes are detected and in case they exist, the usage of method SaveChanges() writes current state to database. If the object is not attached, the changes won't certainly be detected and saved to the database. The object can be attached by using *Attach* method of *DbSet*, but some additional actions are still required because recently attached object has state Unchaged.

The code below shows what is necessary to do.

```csharp
_context.Users.Attach(user);
_context.Entry(user).State = System.Data.EntityState.Modified;
_context.SaveChanges();
```

Bear in mind that if a user with the same Id already exists in ObjectStateManager (for example you successfuly found user with this Id in database before), you will get exception about problem with tracking two entities with the same key. So you can use follow method: _context.Users.**AsNoTracking()**.FirstOrDefault(x => x.Id == user.Id);