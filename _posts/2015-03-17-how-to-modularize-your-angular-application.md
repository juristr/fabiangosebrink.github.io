---
id: 983
title: How to modularize your angular application
date: 2015-03-17 04:43
author: Fabian Gosebrink
layout: post
tags: AngularJS Modules
logo: 'assets/images/logo_small.png'
navigation: True
cover: 'assets/images/aerial-view-of-laptop-notebook-mobile-phone-and-coffee-cup-on-wooden-table.jpg'
subclass: 'post tag-speeches'
disqus: true
categories: blog articles
---

In this blogpost I want to show you a way on how to modularize your angular application in a way I did it so far. Enjoy reading :)

Due to the fact that angular.js gets more and more important you are forced to think about a nice architecture to develop easy. Especially if you develop in mulitple teams.

Well one of the killer features for me in angular is its modularization. Even when you are starting an app you realize that the whole app is nothing else than a module.

{% highlight js %}
var app = angular.module('TestApp',
    [
       //...
    ]);
{% endhighlight %}

Now we could add all our services, and controllers to our defined app. Which would make them all accessable. Behind the scenes angularJS uses dependency injection to offer us our parts of our software and these services are singletons, so they are accessable everywhere out of the box.

![Folie30]({{site.baseurl}}assets/articles/wp-content/uploads/2015/02/Folie30.jpg)

But this is not a nice architecture and we are not using modularization. You do not have an overview of you application when you are using at your application.js file. You are using angularJS to build architectures, so lets do so!

Go and build a folder for every Module you want to create:

![Folders]({{site.baseurl}}assets/articles/wp-content/uploads/2015/02/Folders.png)

And register your services on these modules.

{% highlight js %}
var homeModule = angular.module('home.homeModule', ['ngRoute']);

homeModule.config(function($routeProvider) {
    $routeProvider
        .when("/", {
            controller: "home.controllers.homeController",
            templateUrl: "/app/Home/Templates/overview.html"
        })
        .otherwise({ redirectTo: "/" });
});
{% endhighlight %}

{% highlight js %}
"use strict";
homeModule.controller('home.controllers.homeController', [
    '$scope', 'home.services.peopleService', 'toaster', 'cfpLoadingBar',
    function ($scope, peopleService, toaster, cfpLoadingBar) {

       //...
    }
]);
{% endhighlight %}

Then you have to load your modules in your app like this:

{% highlight js %}
var app = angular.module('TestApp',
    [
        'ngRoute',
        'ngResource',
        'toaster',
        'chieffancypants.loadingBar',

        'home.homeModule',
        'contact.contactModule'
    ]);
{% endhighlight %}


![How to modularize your angular application]({{site.baseurl}}assets/articles/wp-content/uploads/2015/02/Folie31.jpg)

With this you have your application divided in modules and you can add or remove the modules as you want with no effort. You can also easily see which modules your application is based on.

Regards

Fabian
