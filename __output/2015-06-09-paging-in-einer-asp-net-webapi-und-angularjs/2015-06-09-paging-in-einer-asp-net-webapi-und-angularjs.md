---
id: 667
title: Paging in einer ASP.Net WebAPI und AngularJS
date: 2015-06-09T12:14:55+00:00
author: Fabian Gosebrink
layout: post
tags: angular angularjs asp.net 
logo: 'assets/images/logo_small.png'
navigation: True
cover: 'assets/images/download_edit_dark.jpg'
subclass: 'post tag-speeches'
---

# Paging in einer ASP.Net WebAPI (mit AngularJS)

Eine gute API sollte mehrere Features anbieten. Eins davon ist Paging. In diesem Beispiel zeige ich Paging mit AngularJS und wie man es auf dem Client konsumieren kann.

Als erstes sollte man dem Client anbieten per Parameter nur eine bestimmte Anzahl von Einträgen abzurufen.

<pre class="lang:c# decode:true">[Route("myRoute")]
public IHttpActionResult Get(int start = 0, int count = 10)
{
     //...
}</pre>

Man definiert ein Start und eine Anzahl von Items, die abgerufen werden. Per default weisen wir hier zehn Items zu (Range 0-10).

Somit geben wir als Ergebnis per LINQ nur die Anzahl der Items zurück, die der Client angefordert hat.

<pre class="lang:default decode:true">[Route("schedules")]
public IHttpActionResult Get(int start = 0, int count = 10)
{
	try
	{
		IQueryable&lt;MyItem&gt; myItems = _repository.getMyItems();

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
}</pre>

Das Problem an der Stelle ist nun, dass der Client wissen muss, wieviele Items es insgesamt gibt, damit er die richtige Anzahl der Seiten darstellen kann.

Um dem Client die komplette Anzahl der Items mitzuteilen erweitern wir den ResponseHeader um Informationen.

<pre class="lang:default decode:true">[Route("schedules")]
public IHttpActionResult Get(int start = 0, int count = 10)
{
	try
	{
		IQueryable&lt;MyItem&gt; myItems = _repository.getMyItems();
	
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
}</pre>

Im Response-Header steht nun die Anzahl der kompletten Items:

![alttext]({{site.baseurl}}assets/images/blogs/2015-06/80b7bcb0-d187-44ea-8d36-4bce4da4ab3a.png)

Diese brauchen wir dann nur noch im Client parsen bzw. lesen.

Ich benutze die Pagination-Komponente der [Bootstrap-Direktiven](https://angular-ui.github.io/bootstrap/#/pagination).

Html:

<pre class="lang:xhtml decode:true">&lt;pagination 
ng-show="totalItems &gt; maximalItemsPerPage" 
items-per-page="maximalItemsPerPage" 
total-items="totalItems" 
ng-model="currentPage" 
ng-change="pageChanged()"&gt;&lt;/pagination&gt;</pre>

Hierbei wird die Leiste zum navigieren nur angezeigt, wenn die Anzahl der Items grösser ist als die, die maximal auf einer Seite angezeigt werden sollen.

<pre class="lang:js decode:true ">myModule.controller('myController', [
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
</pre>

Hier werden die Standardwerte gesetzt und beim erfolgreichen Abrufen der Items wird der Header ausgelesen und die Variable &#8220;totalItems&#8221; gesetzt, auf die dann gebunden wird.

![alttext]({{site.baseurl}}assets/images/blogs/2015-06/308ca04c-abb0-4292-ae94-45693b5f4832.png)

Grüsse

Fabian

This post was first published on [Noser-Blog](http://blog.noser.com/implementieren-von-paging-in-einer-asp-net-webapi-mitangularjs/)