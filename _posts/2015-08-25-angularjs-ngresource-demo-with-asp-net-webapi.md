---
id: 999
title: AngularJs NgResource Demo with ASP.NET WebAPI
date: 2015-08-25T11:45:45+00:00
author: Fabian Gosebrink
layout: post
tags: github rest webapi ngresourcce
logo: 'assets/images/logo_small.png'
navigation: True
cover: 'assets/images/download_edit_dark.jpg'
subclass: 'post tag-speeches'
disqus: true
categories: articles
---


In this blogpost I want to show a simple AngularJs NgResource Demo with ASP.NET WebAPI querying data as JSON.

Code: <a href="https://github.com/FabianGosebrink/NgResourceDemoApp">https://github.com/FabianGosebrink/NgResourceDemoApp</a>

NgResource: <a href="https://docs.angularjs.org/api/ngResource/service/$resource">https://docs.angularjs.org/api/ngResource/service/$resource</a>

&nbsp;

<h4>The application</h4>

The application follows the "normal" way of seperating the data access also on client side into services.

<a href="http://offering.solutions/wp-content/uploads/2015/08/folders.png"><img class="aligncenter size-full wp-image-1443" src="http://offering.solutions/wp-content/uploads/2015/08/folders.png" alt="AngularJs NgResource Demo with ASP.NET WebAPI" width="237" height="151" /></a>

&nbsp;

<h4>Preparation:</h4>

Before you can use angulars ng resource you have to include it into your application like this:

<pre class="lang:js decode:true ">(function () {
    "use strict";
    angular.module('AngularJsDemoApp',
    [
        'ngRoute',
        'ngAnimate',
        'ngResource',
        "ui.bootstrap",
        'angular-loading-bar',
        "toastr",

        'home.homeModule',
        'contact.contactModule'
    ]).config(['cfpLoadingBarProvider', function (cfpLoadingBarProvider) {
        cfpLoadingBarProvider.includeSpinner = false;
    }]);
}
());</pre>

So here you are providing "ngResource" to your app.

After this you can inject the ngResource service provided by angular into your services:

<pre class="lang:js decode:true ">(function () {
    "use strict";
    angular.module('home.homeModule').factory("home.services.peopleService",
        [
            "$resource", 
            function ($resource) {
                return $resource("api/home/:id", null,
                {
                    'update': {
                        method: 'PUT'
                    }
                });
            }
        ]);
}());</pre>

The first argument you pass in is the url with the optional id (:id). The ng resource service now can call get create and delete methods ist a rest convenient way.

ngresource is providing you the following methods:

<pre class="lang:js decode:true " data-url=" ">{ 
  'get':    {method:'GET'},
  'save':   {method:'POST'},
  'query':  {method:'GET', isArray:true},
  'remove': {method:'DELETE'},
  'delete': {method:'DELETE'} 
};</pre>

&nbsp;

<h4>Usage</h4>

Querying all resources:

<pre class="lang:js decode:true">peopleService.query({},
    function (data) {
        //Success
        vm.allPeople = data;
    },
    function () {
        //Error
    });</pre>

Pay attention to the empty object we are passing in. You could add a single id here if you only want to query one single resource.

<pre class="lang:c# decode:true ">peopleService.query({id: 4},
    function (data) {
        //Success
    },
    function () {
        //Error
    });</pre>

In case you want to add a new resource, a person in this case, you can just call the "save" method and pass in the person you want to save:

<pre class="lang:js decode:true "> peopleService.save(vm.newPerson,
    function (data) {
        //Success
    },
    function (response) {
        //Error
    });</pre>

The service maps the "save"-method to the POST-Action from REST and is executing a POST- call against the API.

&nbsp;

Deleting is self explaining:

<pre class="lang:js decode:true "> peopleService.delete({ id: personToDelete.Id }, function () {
                   // success
    }, function () {
        //Error
    });</pre>

<h4>Conclusion:</h4>

The ngResource provides a good way to interact with a rest api using very very small code but providing big functionality.

&nbsp;

HTH

&nbsp;

Fabian
