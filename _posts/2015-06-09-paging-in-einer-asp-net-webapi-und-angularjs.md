---
id: 978
title: Paging in einer ASP.Net WebAPI und AngularJS
date: 2015-06-09 12:14
author: Fabian Gosebrink
layout: post
tags: AngularJS ASP.NET
logo: 'assets/images/logo_small.png'
navigation: True
cover: 'assets/images/aerial-view-of-laptop-notebook-mobile-phone-and-coffee-cup-on-wooden-table.jpg'
subclass: 'post tag-speeches'
disqus: true
categories: articles
---

In diesem Blogpost will ich zeigen, wie man paging in einer ASP.Net WebAPI und AngularJS realisieren kann.

Eine gute API sollte mehrere Features anbieten. Eins davon ist Paging. In diesem Beispiel zeige ich Paging mit AngularJS und wie man es auf dem Client konsumieren kann.

Als erstes sollte man dem Client anbieten per Parameter nur eine bestimmte Anzahl von Einträgen abzurufen.


{% highlight cs %}
[Route("myRoute")]
public IHttpActionResult Get(int start = 0, int count = 10)
{
     //...
}
{% endhighlight %}

Man definiert ein Start und eine Anzahl von Items, die abgerufen werden. Per default weisen wir hier zehn Items zu (Range 0-10).

Somit geben wir als Ergebnis per LINQ nur die Anzahl der Items zurück, die der Client angefordert hat.

{% highlight cs %}
[Route("schedules")]
public IHttpActionResult Get(int start = 0, int count = 10)
{
    try
    {
        IQueryable<MyItem> myItems = _repository.getMyItems();

        var result = myItems
            .Skip(start)
            .Take(count)
            .ToList();

        return Ok(result);
    }
    catch (Exception)
    {
        return InternalServerError();
    }
}
{% endhighlight %}

Das Problem an der Stelle ist nun, dass der Client wissen muss, wieviele Items es insgesamt gibt, damit er die richtige Anzahl der Seiten darstellen kann.

Um dem Client die komplette Anzahl der Items mitzuteilen erweitern wir den ResponseHeader um Informationen.

{% highlight cs %}
[Route("schedules")]
public IHttpActionResult Get(int start = 0, int count = 10)
{
    try
    {
        IQueryable<MyItem> myItems = _repository.getMyItems();
    
        var paginationHeader = new
        {
            totalCount = myItems.Count(),
        };

        HttpContext.Current.Response.Headers.Add("X-Pagination",
           JsonConvert.SerializeObject(paginationHeader));

        var result = myItems
            .Skip(start)
            .Take(count)
            .ToList();

        return Ok(result);
    }
    catch (Exception)
    {
        return InternalServerError();
    }
}
{% endhighlight %}

Im Response-Header steht nun die Anzahl der kompletten Items:

![Paging in einer ASP.Net WebAPI und AngularJS]({{site.baseurl}}assets/articles/wp-content/uploads/2015/06/1.png)

Diese brauchen wir dann nur noch im Client parsen bzw. lesen.

Ich benutze die Pagination-Komponente der [Bootstrap-Direktiven](https://angular-ui.github.io/bootstrap/#/pagination).

Html:

{% highlight html %}
<pagination 
ng-show="totalItems > maximalItemsPerPage" 
items-per-page="maximalItemsPerPage" 
total-items="totalItems" 
ng-model="currentPage" 
ng-change="pageChanged()"></pagination>
{% endhighlight %}

Hierbei wird die Leiste zum navigieren nur angezeigt, wenn die Anzahl der Items grösser ist als die, die maximal auf einer Seite angezeigt werden sollen.

{% highlight js %}
myModule.controller('myController', [
    '$scope', "myRepository",
    function ($scope, myRepository) {

        $scope.currentPage = 1;
        $scope.maximalItemsPerPage = 5;

    var getMyItems = function (start, count) {
            myRepository.getAllItems(start, count).then(
                function (result) {
                    //Success
                    var totalPagesObject = JSON.parse(result.headers()['x-pagination']);
                    $scope.totalItems = totalPagesObject.totalCount;
                },
                function () {
                    //Error
                });
        };

        //...
     
        $scope.pageChanged = function () {
            console.log('Page changed to: ' + $scope.currentPage);
            getMyItems (($scope.currentPage - 1) * $scope.maximalItemsPerPage, $scope.maximalItemsPerPage);
        };
    }
]);
{% endhighlight %}

Hier werden die Standardwerte gesetzt und beim erfolgreichen Abrufen der Items wird der Header ausgelesen und die Variable "totalItems" gesetzt, auf die dann gebunden wird.

![Paging in einer ASP.Net WebAPI und AngularJS]({{site.baseurl}}assets/articles/wp-content/uploads/2015/06/2.png)

Grüsse

Fabian
