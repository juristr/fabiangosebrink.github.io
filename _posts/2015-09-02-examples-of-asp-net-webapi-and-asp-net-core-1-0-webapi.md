---
id: 735
title: Examples of ASP.Net WebAPI and ASP.Net Core 1.0 WebAPI
date: 2015-09-02T17:04:42+00:00
author: Fabian Gosebrink
layout: post
tags: asp.net mvc6 webarchitecture webapi 
logo: 'assets/images/logo_small.png'
navigation: True
cover: 'assets/images/download_edit_dark.jpg'
subclass: 'post tag-speeches'
disqus: true
categories: articles
---

Hey folks,

I just released two sample APIs on Github.

One of them is done with the "old" approach but heads in the direction of my understanding of a modern WebAPI using OData and having DTOs instead of delivering the plain entitites down to the client.

I will cover the most important information now:

The get method includes odata and is restricting the page-size to 50 to prevent anyone from getting \_all\_ entries which can cause problems. I also added a paginationheader to enable paging and give the information included in the header "X-Pagination" value. Here you can add more stuff like link to the next page, link ot the previous page etc.

I am also using mapper to get a DataTransferObject (DTO) to send it to the client instead of my plain "entity".

```
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
```

The only important thing to mention about the Create-Method is that we are returning a CreatedAtRoute-Actionresult. This causes, that we have a link to the created resource in the header so we can either redirect on the client directly to it, provide the link to the user or work with it in another way.
  
Also take care about sending back the Modelstate in case of an invalid dto.

```
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
```

Mostly forgotten: The Http-Patch-Verb. Its receiving a generic Delta-Object which can be applied to an existing entity and then can be updated.

```
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
```

And last but not least the delete-method. See the fact that one has to return a nocontent (204) which tells the client that the delete was successful.

```
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
```

You can find it here:

https://github.com/FabianGosebrink/ASPNET-WebAPI-Sample

It supports the GET/POST/PUT/PATCH/DELETE, so the normal CRUD-Approach. OData is included.

THe MVC6-Approach is present in another repository.
  
<https://github.com/FabianGosebrink/ASPNET-Core-WebAPI-Sample>

Here OData is not yet included. And the Patch is missing. Lets wait until MVC6 is finally released 😉

Comment if you have suggestions to make these examples better! Let me know.

HTH

Fabian
