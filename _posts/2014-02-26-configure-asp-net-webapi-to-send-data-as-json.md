---
id: 11
title: Configure ASP.Net WebAPI to send data as JSON
date: 2014-02-26T19:51:54+00:00
author: Fabian Gosebrink
categories: articles
layout: post
tags: asp.net configuration json 
logo: 'assets/images/logo_small.png'
navigation: True
cover: 'assets/images/aerial-view-of-laptop-notebook-mobile-phone-and-coffee-cup-on-wooden-table.jpg'
subclass: 'post tag-speeches'
disqus: true
---

Hey,

I just want to show you quickly how to tell ASP.NET WebAPI to send data only as JSON:

ASP.NET WebAPI is normally sending data as XML. The standard data-fomat in Javascript frameworks like Knockout.js or Angular.js is normally JSON. So to get ASP.NET WebAPI using only JSON can be configured in the Register-Method in WebApiConfig.

![alttext]({{site.baseurl}}assets/images/blogs/2014-02-26/d4dbd143-c0e4-461f-a874-903ff24b7e5b.png)

Here in the first part you can see clearing all formatters first and then add the JsonMediaTypeFormatter as the only formatter WebApi knows for my application. This makes sure I am only using JSON-format.

<pre class="lang:c# decode:true ">config.Formatters.Clear();
config.Formatters.Add(new JsonMediaTypeFormatter());</pre>

Fabian
