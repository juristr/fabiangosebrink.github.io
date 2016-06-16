---
id: 505
title: Unit of Work and the Repository Pattern with Entity Framework 0.6.2
date: 2014-09-08T18:41:13+00:00
author: Fabian Gosebrink
layout: post
tags: codefirst development entityframework unitofwork 
logo: 'assets/images/logo_small.png'
navigation: True
cover: 'assets/images/download_edit_dark.jpg'
subclass: 'post tag-speeches'
---

## Unit of Work and the Repository Pattern with Entity Framework 0.6.2

Article to Version 0.6.1 with examples is [here](http://offering.solutions/2014/08/22/nuget-unitofwok-structure-by-offering-solutions/ "[Update] – [NuGet] – UnitOfWork-Structure by Offering.Solutions (with Example-Solution)")

> <span class="repository-meta-content">Offering you a complete abstraction of the UnitOfWork-Pattern with the basic CRUD-Operations, the Repository Pattern and extended functions like CustomRepositores all in one small lib. Made for the Entity Framework. </span>

I just released the Unit of Work and the Repository Pattern with Entity Framework in Version 0.6.2.

&nbsp;

<h3 style="text-align: center;">
  <a href="https://www.nuget.org/packages/OfferingSolutions.UnitOfWork.Structure/0.6.2">Nuget: UnitOfWork by Offering.Solutions 0.6.2</a>
</h3>

&nbsp;

Changes:

  * Added &#8220;DatabaseContext&#8221; insted of &#8220;DbContext&#8221; to the CustomRepositories in ExampleSolution &#8211;> No need to call &#8220;Set<T>&#8221;
  * Made basic CRUD methods virtual so you can overwrite them easier

<!--more-->

<pre class="lang:c# decode:true ">public class PersonRepository : RepositoryContextImpl&lt;Person&gt;, IPersonRepository
{
	public PersonRepository(DatabaseContext dbContext)
		: base(dbContext)
	{

	}

	public void MyNewFunction(int id)
	{
		//Do Something
	}

	public override void Add(Person toAdd)
	{
		MyAdditionalAddFunction();
		base.Add(toAdd);
	}

	private void MyAdditionalAddFunction()
	{
		//Do something else...
	}
}</pre>

Download [OfferingSolutions.UnitOfWork.Sample.0.6.2](http://offering.solutions/wp-content/uploads/2014/09/OfferingSolutions.UnitOfWork.Sample.0.6.2.zip)

You can also see this Project on Github here: [OfferingSolutions Repository Pattern UnitOfWork](https://github.com/OfferingSolutions/OfferingSolutions-RepositoryPattern-UnitOfWork)

Regards

Fabian
