---
id: 975
title: AngularJS NgResource Demo with ASP.NET WebAPI
date: 2015-08-25 11:45
author: Fabian Gosebrink
layout: post
tags: AngularJS Github JSON ngresource WebAPI ASP.NET
logo: 'assets/images/logo_small.png'
navigation: True
cover: 'assets/images/aerial-view-of-laptop-notebook-mobile-phone-and-coffee-cup-on-wooden-table.jpg'
subclass: 'post tag-speeches'
disqus: true
categories: articles
---

In this blogpost I want to show a simple AngularJS NgResource Demo with ASP.NET WebAPI querying data as JSON.

Code: [https://github.com/FabianGosebrink/NgResourceDemoApp](https://github.com/FabianGosebrink/NgResourceDemoApp)

NgResource: [https://docs.AngularJS.org/api/ngResource/service/$resource](https://docs.AngularJS.org/api/ngResource/service/$resource)

#### The application

The application follows the "normal" way of seperating the data access also on client side into services.

![AngularJS NgResource Demo with ASP.NET WebAPI]({{site.baseurl}}assets/articles/wp-content/uploads/2015/08/folders.png)

#### Preparation:

Before you can use angulars ng resource you have to include it into your application like this:

{% highlight js %}
(function () {
    "use strict";
    angular.module('AngularJSDemoApp',
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
}());
{% endhighlight %}

So here you are providing "ngResource" to your app.

After this you can inject the ngResource service provided by angular into your services:

{% highlight js %}
(function () {
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
}());
{% endhighlight %}

The first argument you pass in is the url with the optional id (:id). The ng resource service now can call get create and delete methods ist a rest convenient way.

ngresource is providing you the following methods:

{% highlight js %}
{ 
  'get':    {method:'GET'},
  'save':   {method:'POST'},
  'query':  {method:'GET', isArray:true},
  'remove': {method:'DELETE'},
  'delete': {method:'DELETE'} 
};
{% endhighlight %}

#### Usage
    
Querying all resources:

{% highlight js %}
peopleService.query({},
    function (data) {
        //Success
        vm.allPeople = data;
    },
    function () {
        //Error
    });
{% endhighlight %}

Pay attention to the empty object we are passing in. You could add a single id here if you only want to query one single resource.

{% highlight js %}
peopleService.query({id: 4},
    function (data) {
        //Success
    },
    function () {
        //Error
    });
{% endhighlight %}

In case you want to add a new resource, a person in this case, you can just call the "save" method and pass in the person you want to save:

{% highlight js %}
peopleService.save(vm.newPerson,
    function (data) {
        //Success
    },
    function (response) {
        //Error
    });
{% endhighlight %}

The service maps the "save"-method to the POST-Action from REST and is executing a POST- call against the API.

Deleting is self explaining:

{% highlight js %}
peopleService.delete({ id: personToDelete.Id }, 
    function () {
        // success
    }, function () {
        //Error
    });
{% endhighlight %}

#### Conclusion:

The ngResource provides a good way to interact with a rest api using very very small code but providing big functionality.

HTH

Fabian
