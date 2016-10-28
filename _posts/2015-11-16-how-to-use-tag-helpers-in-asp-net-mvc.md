---
id: 817
title: How to use tag helpers in ASP.NET MVC
date: 2015-11-16-16T20:18:34+00:00
author: Fabian Gosebrink
layout: post
tags: asp.net asp.netmvc 
logo: 'assets/images/logo_small.png'
navigation: True
cover: 'assets/images/download_edit_dark.jpg'
subclass: 'post tag-speeches'
disqus: true
categories: articles
---

Hey,

in this post I want to show you how to use tag helpers in ASP.NET MVC 6.

First of all: What are the tag helpers good for?

If you are coding your links in your MVC-Views with normal strings its really hard to refactor or correct them in case you rename your routing, controller or your action. So you need a kind of generic approach to do this.

Razor in previous versions had this

```
Url.Action("MyAction", "MyController", new { id = "123" })
```

which returned an url like "MyController/MyAction/123"

MVC 6 is now introducing a new more readable way to achieve the same result called TagHelpers.

> If you are using the normal MVC6-Template from Visual Studio asp-net TagHelpers are already included! However, I will mention the necessary steps anyway.

Getting started:

First make sure you included a reference to the tag helpers in your project.json

![How to use tag helpers in ASP.NET MVC]({{site.baseurl}}assets/articles/2015-11-16/bc1c8d13-1a2a-4e9c-b8c9-4d21ae512b93.png)

And Because tag helpers do not throw an exception if you use them and they do not work you should also include a _ViewImports.cshtml which is responsible to load all extra functionality to your views. You can simply add it via the context menu Add &#8211;> New Item.

![How to use tag helpers in ASP.NET MVC]({{site.baseurl}}assets/articles/2015-11-16/7ef2ff9d-2743-4bbd-8c80-74f734df4dbb.png) 
![How to use tag helpers in ASP.NET MVC]({{site.baseurl}}assets/articles/2015-11-16/dbbe329a-4671-4e15-84c9-f417e79a8741.png)

Now all the cshtml files are able to handle Asp.Net-TagHelpers which get easily in your html like this:

```
<li><a asp-controller="Home" asp-action="Contact">Contact</a></li>
```

which is rendered to

```
<li><a href="/Home/Contact">Contact</a></li>
```

That is great because you do not have to use a non-html-syntax anymore and it it easier to understand and to read.

You can also write forms in this Html-Tag-Helper-Syntax which is really nice to understand

```
<form asp-controller="Home" asp-action="SendData" method="post" class="form-horizontal" role="form">
    <div asp-validation-summary="ValidationSummary.All" class="text-danger"></div>
    <div class="form-group">
        <label asp-for="Name" class="col-md-2 control-label"></label>
        <div class="col-md-10">
            <input asp-for="Name" class="form-control" />
            <span asp-validation-for="Name" class="text-danger"></span>
        </div>
    </div>
    <div class="form-group">
        <label asp-for="Age" class="col-md-2 control-label"></label>
        <div class="col-md-10">
            <input asp-for="Age" class="form-control" />
            <span asp-validation-for="Age" class="text-danger"></span>
        </div>
    </div>
    <div class="form-group">
        <div class="col-md-offset-2 col-md-10">
            <button type="submit" class="btn btn-default">Send person</button>
        </div>
    </div>
</form>
```

instead of using the "ugly" @-like Sytax before.

Hope to make you curious about tag helpers. Go and use them in your next ASP.NET-Project

Regards

Fabian
