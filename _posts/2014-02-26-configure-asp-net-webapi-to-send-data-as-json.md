---
id: 11
title: Configure ASP.NET WebAPI to send data as JSON
date: 2014-02-26T19:51:54+00:00
author: Fabian Gosebrink
categories: blog articles
layout: post
tags: asp.net configuration json 
logo: 'assets/images/logo_small.png'
navigation: True
cover: 'assets/images/aerial-view-of-laptop-notebook-mobile-phone-and-coffee-cup-on-wooden-table.jpg'
subclass: 'post tag-speeches'
disqus: true
---

Hey,

In this blogpost I just want to show you quickly how to configure ASP.NET WebAPI to send data as JSON

ASP.NET WebAPI is normally sending data as XML. The standard data-fomat in Javascript frameworks like Knockout.js or Angular.js is normally JSON. So to get ASP.NET WebAPI using only JSON can be configured in the Register-Method in WebApiConfig.

![Configure ASP.NET WebAPI to send data as JSON]({{site.baseurl}}assets/articles/2014-02-26/d4dbd143-c0e4-461f-a874-903ff24b7e5b.png)

Here in the first part you can see clearing all formatters first and then add the JsonMediaTypeFormatter as the only formatter WebApi knows for my application. This makes sure I am only using JSON-format.

{% highlight csharp %}
config.Formatters.Clear();
config.Formatters.Add(new JsonMediaTypeFormatter());
{% endhighlight %}

If you now sending requests to the API it should return the data as json.

<h4>Complete sample Startup:</h4>

{% highlight csharp %}
var config = new HttpConfiguration();

config.Formatters.Clear();
config.Formatters.Add(new JsonMediaTypeFormatter());

config.MapHttpAttributeRoutes();

config.Routes.MapHttpRoute(
    name: "DefaultApi",
    routeTemplate: "api/{controller}/{id}",
    defaults: new { id = RouteParameter.Optional }
);

app.UseWebApi(config);
{% endhighlight %}

Consider that if a client is sending the Accept-Header in the request with a value your API does not support it will get no data. Although JSON is the de facto standard for REST-APIs you should always remember to have your API as easy and intuitive as possible to consume for your clients. So maybe if your API goes public think about having XML and JSON as formatters.

HTH

Fabian
