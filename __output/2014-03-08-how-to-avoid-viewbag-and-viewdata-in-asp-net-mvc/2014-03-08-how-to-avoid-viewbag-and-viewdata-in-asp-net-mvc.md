---
id: 70
title: How to avoid ViewBag and ViewData in ASP.Net MVC
date: 2014-03-08T13:24:01+00:00
author: Fabian Gosebrink
layout: post
tags: asp.netmvc viewbag viewdata 
logo: 'assets/images/logo_small.png'
navigation: True
cover: 'assets/images/download_edit_dark.jpg'
subclass: 'post tag-speeches'
---

# How to avoid ViewBag and ViewData in ASP.Net MVC

It could be so easy: When passing data from the Controller to the View in MVC one can simply use

<pre>ViewBag.MyProperty = “ThisIsTheContentOfMyProperty”;</pre>

or

<pre>ViewData["MyProperty"] = MyProperty;</pre>

And in the view you can access the data with:

<!--more-->

<pre>@ViewBag.MyProperty</pre>

or

<pre>ViewData["MyProperty "] as ...</pre>

But what about spelling problems? _IntelliSense_ will not correct you if you would miss a character. Even the compiler does not give you any hint.

<pre>@ViewBag.MProperty</pre>

would not be wrong but won’t show any data in your View. Also spelling problems in the ViewData-String would not be noticed in code.

In general: Using the MVC-Pattern is great. So when Asp.net **MVC** gives us the possibility to use this pattern: Do so!

A Viewbag (also Viewdata, Viewbag is only a wrapper around Viewdata) can be used like a bucket for your data. But this is not nice and it’s harming the Mvc-Pattern!

The view knows its model and should not get any data from anything else. So to avoid using any pails for your data, use ViewModels to pass your data into the View.

This could look like this:

<pre escaped="true">public class MyViewModel

{

public List MyModels { get; set; }

public string Rooms { get; set; }

public bool IsSomethingTrueOrNot { get; set; }

...

}</pre>

&nbsp;

And in the View you can simply pass the complete ViewModel to your View

<pre>public ActionResult MyMethod()

{

MyViewModel viewModel = new MyViewModel();

// Do anything with the ViewModel like filling it, etc.

return View(viewModel);

}</pre>

&nbsp;

With this simple technique you can keep all the data you need for creating a view separately from your MVC-Models.

This can be tested well; it can be simply used and gives you more structure arrangement to your MVC-Projects.

It’s so easy, isn’t it?

Happy coding!
