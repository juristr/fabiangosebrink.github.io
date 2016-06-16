---
id: 588
title: How to modulize your angular application
date: 2015-03-17T04:43:54+00:00
author: Fabian Gosebrink
layout: post
tags: 
logo: 'assets/images/logo_small.png'
navigation: True
cover: 'assets/images/download_edit_dark.jpg'
subclass: 'post tag-speeches'
---

## How to modulize your angular application

Due to the fact that angular.js gets more and more important you are forced to think about a nice architecture to develop easy. Especially if you develop in mulitple teams.

Well one of the killer features for me in angular is its modularization. Even when you are starting an app you realize that the whole app is nothing else than a module.

&nbsp;

<pre class="lang:js decode:true ">var app = angular.module('TestApp',
    [
       //...
    ]);</pre>

Now we could add all our services, and controllers to our defined app. Which would make them all accessable. Behind the scenes angularJS uses dependency injection to offer us our parts of our software and these services are singletons, so they are accessable everywhere out of the box.

![alttext]({{site.baseurl}}assets/images/blogs/2015-02/42841aaf-195a-43a5-8562-ec9541edddba.jpg)

But this is not a nice architecture and we are not using modularization. You do not have an overview of you application when you are using at your application.js file. You are using angularJS to build architectures, so lets do so!

Go and build a folder for every Module you want to create:

![alttext]({{site.baseurl}}assets/images/blogs/2015-02/95b7dcaf-644a-43cf-920b-45786520bae6.png)And register your services on these modules.

<pre class="lang:js decode:true ">var homeModule = angular.module('home.homeModule', ['ngRoute']);

homeModule.config(function($routeProvider) {
    $routeProvider
        .when("/", {
            controller: "home.controllers.homeController",
            templateUrl: "/app/Home/Templates/overview.html"
        })
        .otherwise({ redirectTo: "/" });
});</pre>

&nbsp;

<pre class="lang:js decode:true ">"use strict";
homeModule.controller('home.controllers.homeController', [
    '$scope', 'home.services.peopleService', 'toaster', 'cfpLoadingBar',
    function ($scope, peopleService, toaster, cfpLoadingBar) {

       //...
    }
]);

</pre>

Then you have to load your modules in your app like this:

<pre class="lang:js decode:true">var app = angular.module('TestApp',
    [
        'ngRoute',
        'ngResource',
        'toaster',
        'chieffancypants.loadingBar',

        'home.homeModule',
        'contact.contactModule'
    ]);
</pre>

![alttext]({{site.baseurl}}assets/images/blogs/2015-02/10f82ac7-1f0c-43c6-bb4f-b387928aafe6.jpg)

With this you have your application divided in modules and you can add or remove the modules as you want with no effort. You can also easily see which modules your application is based on.

&nbsp;

Regards

&nbsp;

Fabian
