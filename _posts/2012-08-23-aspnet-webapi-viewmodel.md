---
layout: post
title: "ASP.NET WebAPI ViewModel"
description: "It is quite often objects, which are aimed to transfer a data between a client and a server or to represent the objects of the real world, are different from data-access layer or buisness logic objects. In this case it is recommended to use DTO. Concerning ASP.NET WebAPI that designed by MVC pattern DTO can be used as a view model in client-server interaction. There are recommendations for ASP.NET MVC to use View Model object to transfer information to View instead of Entity Framework enities for example, and for WebAPI it is more actual. For convertion Model to View Model and back it is very convenient to use AutoMapper."
category: dev
tags: ["DTO", "ASP.NET WebAPI", "AutoMapper"]
---
{% include JB/setup %}

It is quite often objects, which are aimed to transfer a data between a client and a server or to represent the objects of the real world, are different from data-access layer or buisness logic objects. In this case it is recommended to use [DTO][dto]. Concerning ASP.NET WebAPI that designed by MVC pattern DTO can be used as a view model in client/server interaction. There are recommendations for ASP.NET MVC to use View Model object to transfer information to View instead of Entity Framework enities for example, and for WebAPI it is more actual. For convertion Model to View Model and back it is very convenient to use [AutoMapper][am]. NuGet easily allows to add this library to the project (type into your Package manager console: `Install-Package AutoMapper`).

To use AutoMapper it is necessery to configure how the objects of one type will be mapped to once of another type. If we use model and DTO object that have the same names of properties, the configuration will take ony one line of code. If we have a more complex object that requires more complex convertion, we can use a fluent-like configuration or inheritance from `TypeConverter<srcT, destT>` and realization of its abstract method `destT ConvertCore(srcT source)`. Here is the example:

```csharp
Mapper.CreateMap<SimpleModel, SimpleViewModel>();

Mapper.CreateMap<Model, ViewModel>()
        .ConvertUsing<ModelToViewModelConverter>();
```

And realization of converter:

```csharp
public class ModelToViewModelConverter 
    : TypeConverter<Model, ViewModel> {

    protected override ViewModel ConvertCore(Model source) {

        var destination = new ViewModel {
            Name = source.Name,
            Total = source.Balance + source.Incomes.Sum(x => x.Value)
        };
        destination.Initialize();

        return output;
    }
}
```

The mapping from model to view model in ApiController's method might be implemented directly in the body of method by the invocation of generic-function Map or creating our own ActionFilterAttribute for methods of ApiController. And this action filter will perform mapping of objects. I'm going to show the second way and present the code of controller's method below as result.

```csharp
[AutoMap(typeof(ViewModel))]                         
public Model Get(Guid id) {                           
    var model = _modelRepository.GetById(id);        
    if (model == null)                               
        return GetException(HttpStatusCode.NotFound);
                                                     
    return model;                                    
}
```

To create our own ActionFilterAttribute, we need to inherit from `ActionFilterAttribute` from **System.Web.Http.Filters** namespace (but not System.Web.Mvc.Filters) and override method **OnActionExecuted** as it shown below. 

```csharp
[AttributeUsage(AttributeTargets.Method)]
public class AutoMapAttribute : ActionFilterAttribute {
    /// <summary>
    /// Action filter converts model to view model using AutoMapper. 
    /// </summary>
    public AutoMapAttribute(Type convertTo) {
        ConvertTo = convertTo;
    }
    /// <summary>
    /// Type of view model.
    /// </summary>
    public Type ConvertTo { get; set; }

    public override void OnActionExecuted(HttpActionExecutedContext actionContext) {
        HttpStatusCode status = actionContext.Response.StatusCode;
        // check that result of action is positive
        bool isNegativeResponse =
               status != HttpStatusCode.Created
            && status != HttpStatusCode.OK;
        if (isNegativeResponse) {
            return;
        }
        // get model from response message
        object model;
        if (!actionContext.Response.TryGetContentValue(out model)) {
            actionContext.Response = new HttpResponseMessage(HttpStatusCode.InternalServerError);
            return;
        }
        // convert model to view model and set new response to action context
        var viewModel = Mapper.Map(model, model.GetType(), ConvertTo);
        actionContext.Response = actionContext.Request.CreateResponse(status, viewModel);
    }
}
```

Let's pay attention to the method OnActionExecuted. First of all, we should check the status of server response. If it is positive we try to get model from the response and if we have got success we are going to use AutoMapper to transfom it into ViewModel. Here we use other overriding of the method Map instead of generic-method. Also we have to pass the type of mapped object second by the second argument for correct detection of model type. It is necessary because we got the object of model from the context of Entity Framwork, and AutoMapper detects its type as type of proxy-object and does not find suitable mapping.

That's all.

[dto]: http://martinfowler.com/eaaCatalog/dataTransferObject.html
[am]: http://automapper.codeplex.com/