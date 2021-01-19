---
layout: post
title: "Blazor - Expression"
date: 2021-01-19 09:10:13
background: '/assets/img/posts01.jpg'
---

In this post, we're going to learn about the _secret_ feature in Blazor that not a lot of people know exists. On its own, it's not very useful but it can be used in some advanced scenarios, for example for implementing custom validations.

## Introduction

Most of you already know about how Blazor two-way binding works. In case you don't, just google it. I believe there are already many sources around explaining it in detail. In this post, I will just go over it very quickly.

Usually, when we want to use a component we use it like this:

```cs
<TextField @bind-Value="@name">

@code{
    string name = "John";
}
```

Notice the `@bind-Value` parameter. When we write it like that it's called a two-way binding. Meaning whenever a `Value` changes it will also update the `name` field. It also works the other way. When `name` changes, it will also update `TextField` internal value. Magic!

So how it works?

Behind the scenes, in `TextField` we must have two parameters. A `Value` and `ValueChanged`. Whenever we have a `PropertyName` and `[PropertyName]Changed`, Blazor will have enough information on how to do the two-way binding.

### Example

```cs
// other code

[Parameter] public string ValueChanged { get; set; }
[Parameter] public EventCallback<string> ValueChanged { get; set; }
```

And now that we learned about two-way binding I'm going to teach you about its secret feature, an expression getter.

## Expression

 In our previous example, we only tried to modify a `name` field. But what if we want to know more about that same field? What if we add a special attribute to it and want to get that information?

```cs
<TextField @bind-Value="@name">

@code{
    [MySpecialAttribute]
    string name = "John";
}
```

The solution if quite simple. We only need to add a third parameter to our `TextField` component.

```cs
[Parameter] public string ValueChanged { get; set; }
[Parameter] public EventCallback<string> ValueChanged { get; set; }

[Parameter] public Expression<Func<string>> ValueExpression { get; set; }
```

Now we can use `ValueExpression` to get all the information. You can use [expression tree](https://www.tutorialsteacher.com/linq/expression-tree) to get access to the `name` attribute. Or, in case you want to implement your own validation as I did for [Blazorise](https://github.com/stsrki/Blazorise) you can use like this


```cs
void SomeMethod()
{
     var fieldIdentifier = FieldIdentifier.Create( ValueExpression );

    Console.WriteLine($"Model: {fieldIdentifier.Model} Field: {fieldIdentifier.FieldName}");

     // other code
}
```

> For a full validation source-code have a look at the [Blazorise](https://github.com/stsrki/Blazorise). It's fully open-source.

Basically, whatever you do with the expression parameter is up to your imagination and your use-cases. Now that you know about it, the sky is the limit. 🚀