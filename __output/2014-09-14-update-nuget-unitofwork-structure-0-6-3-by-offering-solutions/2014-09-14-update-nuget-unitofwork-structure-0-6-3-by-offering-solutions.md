---
id: 527
title: '[Update] – [NuGet] – UnitOfWork-Structure 0.6.3 by Offering.Solutions &#8211; with example solution'
date: 2014-09-14T13:04:45+00:00
author: Fabian Gosebrink
layout: post
tags: codefirst development entityframework unitofwork 
logo: 'assets/images/logo_small.png'
navigation: True
cover: 'assets/images/download_edit_dark.jpg'
subclass: 'post tag-speeches'
---

Article to Version 0.6.2 with examples is [here](http://offering.solutions/2014/09/08/update-nuget-unitofwork-structure-0-6-2-by-offering-solutions-with-example-solution/ "[Update] – [NuGet] – UnitOfWork-Structure 0.6.2 by Offering.Solutions (with Example-Solution)")

&nbsp;

I just released the UnitOfWork by Offering.Solutions in <del>Version 0.6.3.</del> Version 0.7.0 <del><br /> </del>

&nbsp;

<h3 style="text-align: center;">
  <a href="https://www.nuget.org/packages/OfferingSolutions.UnitOfWork.Structure/0.7.0" target="_blank">Nuget: UnitOfWork by Offering.Solutions 0.7.0</a>
</h3>

<h3 style="text-align: center;">
  <a href="http://offering.solutions/2014/12/13/update-nuget-unitofwork-structure-0-7-0-by-offering-solutions/" target="_blank">BlogPost</a>
</h3>

&nbsp;

&nbsp;

Changes:

  * Added async methods to load and save data asynchronously
  * Save has now a return value (int)

<!--more-->

&nbsp;

<pre class="lang:c# decode:true ">private async void PerformDatabaseOperations()
{
	using (IPersonRepository personRepository = new PersonRepository(new DatabaseContext()))
	{
		personRepository.Add(new Person());
		personRepository.Save();
		List&lt;Person&gt; persons = await personRepository.GetAllASync();

		Console.WriteLine(persons.Count);
		personRepository.MyNewFunction(6);
		await personRepository.SaveASync();
		//...
	}
}</pre>

The Sample-Solution of 0.6.2 is still valid

[OfferingSolutions.UnitOfWork.Sample.0.6.2](http://offering.solutions/wp-content/uploads/2014/09/OfferingSolutions.UnitOfWork.Sample.0.6.2.zip)
