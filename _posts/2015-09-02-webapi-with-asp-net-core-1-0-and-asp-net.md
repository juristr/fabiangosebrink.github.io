---
id: 973
title: WebAPI with ASP.NET Core 1.0 and ASP.NET
date: 2015-09-02 17:04
author: Fabian Gosebrink
layout: post
tags: ASP.NET web architecture WebAPI
logo: 'assets/images/logo_small.png'
navigation: True
cover: 'assets/images/aerial-view-of-laptop-notebook-mobile-phone-and-coffee-cup-on-wooden-table.jpg'
subclass: 'post tag-speeches'
disqus: true
categories: blog articles
---

I just released two sample APIs on Github: WebAPI with ASP.NET Core 1.0 and ASP.NET

One of them is done with the "old" approach but heads in the direction of my understanding of a modern WebAPI using OData and having DTOs instead of delivering the plain entitites down to the client.

I will cover the most important information now:

The get method includes odata and is restricting the page-size to 50 to prevent anyone from getting *all* entries which can cause problems. I also added a paginationheader to enable paging and give the information included in the header "X-Pagination" value. Here you can add more stuff like link to the next page, link ot the previous page etc.

I am also using mapper to get a DataTransferObject (DTO) to send it to the client instead of my plain "entity".

{% highlight cs %}
[HttpGet]
[EnableQuery(PageSize = 50)]
public IHttpActionResult Get()
{
    var paginationHeader = new
    {
        totalCount = Singleton.Instance.Houses.Count
        // Add more headers here if you want...
    };

    HttpContext.Current.Response.AppendHeader("X-Pagination", JsonConvert.SerializeObject(paginationHeader));

    return Ok(Singleton.Instance.Houses.Select(x => _houseMapper.MapToDto(x)));
}
{% endhighlight %}

The only important thing to mention about the Create-Method is that we are returning a CreatedAtRoute-Actionresult. This causes, that we have a link to the created resource in the header so we can either redirect on the client directly to it, provide the link to the user or work with it in another way.
Also take care about sending back the Modelstate in case of an invalid dto.

{% highlight cs %}
[HttpPost]
public IHttpActionResult Create([FromBody] HouseDto houseDto)
{
    if (houseDto == null)
    {
        return BadRequest();
    }

    if (!ModelState.IsValid)
    {
        return BadRequest(ModelState);
    }

    HouseEntity houseEntity = _houseMapper.MapToEntity(houseDto);

    Singleton.Instance.Houses.Add(houseEntity);

    return CreatedAtRoute("DefaultApi", new { id = houseEntity.Id }, _houseMapper.MapToDto(houseEntity));
}
{% endhighlight %}

Mostly forgotten: The Http-Patch-Verb. Its receiving a generic Delta-Object which can be applied to an existing entity and then can be updated.

{% highlight cs %}
[HttpPatch]
[Route("{id:int}")]
public IHttpActionResult Patch(int id, Delta<HouseDto> houseDto)
{
    if (houseDto == null)
    {
        return BadRequest();
    }

    if (!ModelState.IsValid)
    {
        return BadRequest(ModelState);
    }

    HouseEntity houseEntityToUpdate = Singleton.Instance.Houses.FirstOrDefault(x => x.Id == id);

    if (houseEntityToUpdate == null)
    {
        return NotFound();
    }

    HouseDto existingHouse = _houseMapper.MapToDto(houseEntityToUpdate);
    houseDto.Patch(existingHouse);

    int index = Singleton.Instance.Houses.FindIndex(x => x.Id == id);
    Singleton.Instance.Houses[index] = _houseMapper.MapToEntity(existingHouse);

    return Ok(existingHouse);
}
{% endhighlight %}

And last but not least the delete-method. See the fact that one has to return a nocontent (204) which tells the client that the delete was successful.

{% highlight cs %}
[HttpDelete]
[Route("{id:int}")]
public IHttpActionResult Delete(int id)
{
    HouseEntity houseEntityToDelete = Singleton.Instance.Houses.FirstOrDefault(x => x.Id == id);

    if (houseEntityToDelete == null)
    {
        return NotFound();
    }

    Singleton.Instance.Houses.Remove(houseEntityToDelete);

    return StatusCode(HttpStatusCode.NoContent);
}
{% endhighlight %}

You can find it here:

[https://github.com/FabianGosebrink/ASPNET-WebAPI-Sample](https://github.com/FabianGosebrink/ASPNET-WebAPI-Sample)

It supports the GET/POST/PUT/PATCH/DELETE, so the normal CRUD-Approach. OData is included.

The ASP.NET Core-approach is present in another repository.
[https://github.com/FabianGosebrink/ASPNET-Core-WebAPI-Sample](https://github.com/FabianGosebrink/ASPNET-Core-WebAPI-Sample)

Comment if you have suggestions to make these examples better! Let me know.

HTH

Fabian
