---
id: 662
title: ASP.NET Datashaping (GitHub)
date: 2015-05-30T18:23:00+00:00
author: Fabian Gosebrink
layout: post
tags: datashaping github rest webapi 
logo: 'assets/images/logo_small.png'
navigation: True
cover: 'assets/images/download_edit_dark.jpg'
subclass: 'post tag-speeches'
disqus: true
categories: articles
---

With this blogpost I want to introduce you the ASP.NET WebAPI Datashaper to strip your json data before sending it back to the client.

The Datashaper gives you the possibility to strip the data you want to send based on the query of the client if you can not use Odata for a reason.

<a href="https://github.com/OfferingSolutions/OfferingSolutions-Datashaper">GitHub</a>
<a href="http://fabian-gosebrink.de/Projects/Datashaper">Project-Page</a>
<a href="http://www.nuget.org/packages/OfferingSolutions.DataShaper/">NuGet</a>

<a href="https://github.com/OfferingSolutions/OfferingSolutions-Datashaper-Demo">Demo</a>

With this Nuget you can add the fields you want to receive in your request like:

<pre class="lang:c# decode:true ">GET /api/test?fields=Id,Title,Date</pre>

or

<pre class="lang:c# decode:true ">GET /api/test?fields=Id,Title,Date,ChildClasses.Description,ChildClasses.Id ...</pre>

You only have to call

<pre class="lang:c# decode:true ">Datashaper.CreateDataShapedObject(x, listOfFields)</pre>

in the end which is going to apply the list of Perperties to your data.

&nbsp;

This can be useful if you wnat to display a table of your data with only selected fields. You do not have to have every property from your model onto the client and display it. You only need specific fields which you can strip out with this package.

This is equivalent to the OData "$select"-Query option. But this package gives you the opportunity to get the same behaviour without using OData. But if you are interested you should take a look onto <a href="http://www.odata.org/">Odata</a>, too.

&nbsp;

Here is an example

```
[Route("myroute")]
public IHttpActionResult Get(string fields = null)
{
    try
    {
        //...
        
        List<string> listOfFields = new List<string>();
        if (fields != null)
        {
            listOfFields = fields.Split(',').ToList();
        }

        IQueryable<MyItems> myItems = _repository.GetMyItems();

        //...
        
        var result = myItems
            .ToList()
            .Select(x => Datashaper.CreateDataShapedObject(x, listOfFields));

        return Ok(result);
    }
    catch (Exception)
    {
         return InternalServerError();
    }
}
```

<a href="http://offering.solutions/wp-content/uploads/2015/05/datashaper_1.png"><img class="aligncenter wp-image-1525 size-full" src="http://offering.solutions/wp-content/uploads/2015/05/datashaper_1.png" alt="ASP.NET WebAPI Datashaper to strip your json data" width="794" height="851" /></a> <a href="http://offering.solutions/wp-content/uploads/2015/05/datashaper_2.png"><img class="aligncenter wp-image-1526 size-full" src="http://offering.solutions/wp-content/uploads/2015/05/datashaper_2.png" alt="ASP.NET WebAPI Datashaper to strip your json data" width="841" height="318" /></a> <a href="http://offering.solutions/wp-content/uploads/2015/05/datashaper_3.png"><img class="aligncenter wp-image-1527 size-full" src="http://offering.solutions/wp-content/uploads/2015/05/datashaper_3.png" alt="ASP.NET WebAPI Datashaper to strip your json data" width="853" height="912" /></a> <a href="http://offering.solutions/wp-content/uploads/2015/05/datashaper_4.png"><img class="aligncenter wp-image-1528 size-full" src="http://offering.solutions/wp-content/uploads/2015/05/datashaper_4.png" alt="ASP.NET WebAPI Datashaper to strip your json data" width="841" height="624" /></a>

&nbsp;

Regards & HTH

Fabian
