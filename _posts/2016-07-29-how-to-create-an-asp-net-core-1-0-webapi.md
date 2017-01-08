---
title: How to create an ASP.NET Core 1.0 WebAPI
date: 2016-07-29 18:31
author: Fabian Gosebrink
layout: post
tags: ASP.NET Core WebAPI
logo: 'assets/images/logo_small.png'
navigation: True
cover: 'assets/images/aerial-view-of-laptop-notebook-mobile-phone-and-coffee-cup-on-wooden-table.jpg'
subclass: 'post tag-speeches'
disqus: true
categories: articles
---

In this blogpost I want to show you how to create an ASP.NET Core 1.0 WebAPI. We will use a custom mapper (you could use AutoMapper for this instead, a repository which is a singleton in this blog here and we will use the normal CRUD operations to Create, Read, Update and Delete in the "database" here.

Code here [https://github.com/FabianGosebrink/ASPNET-Core-WebAPI-Sample](https://github.com/FabianGosebrink/ASPNET-Core-WebAPI-Sample)

### Visual Studio

![AspNetCoreWebApi0]({{site.baseurl}}assets/articles/wp-content/uploads/2016/07/AspNetCoreWebApi0.jpg)
![AspNetCoreWebApi02]({{site.baseurl}}assets/articles/wp-content/uploads/2016/07/AspNetCoreWebApi02.jpg)

We will start with the ASP.NET Startup-File

### ASP.NET Core Startup

{% highlight cs %}
public class Startup
{
    public Startup(IHostingEnvironment env)
    {
        var builder = new ConfigurationBuilder()
           .SetBasePath(env.ContentRootPath)
           .AddJsonFile("appsettings.json", optional: true, reloadOnChange: true)
           .AddJsonFile($"appsettings.{env.EnvironmentName}.json", optional: true);

        builder.AddEnvironmentVariables();
        Configuration = builder.Build();
    }

    public IConfigurationRoot Configuration { get; set; }

    // This method gets called by the runtime. Use this method to add services to the container.
    public void ConfigureServices(IServiceCollection services)
    {
        // Add framework services.
        services.AddMvc();
    }

    // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
    public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
    {
        loggerFactory.AddConsole(Configuration.GetSection("Logging"));
        loggerFactory.AddDebug();

        app.UseDefaultFiles();
        app.UseStaticFiles();

        app.UseMvc();
    }
}
{% endhighlight %}

No magic here so far. We are creating a configuration in the first step (constructor) of the FIle and then add and use MVC with its defaultfiles (e.g. index.html) and add the ability to server static files in general (app.useStaticFiles).

### The models

First lets build some entities we want to send to the client and back. We will create a DTO and an Entity and map everything before it gets send to the client.

{% highlight cs %}
public class HouseDto
    {
        public int Id { get; set; }

        [Required, MinLength(3)]
        public string Street { get; set; }

        [Required, MinLength(3)]
        public string City { get; set; }

        [Required]
        [DataType(DataType.PostalCode)]
        public int ZipCode { get; set; }
}
{% endhighlight %}

{% highlight cs %}
public class HouseEntity
    {
        public int Id { get; set; }
        public string Street { get; set; }
        public string City { get; set; }
        public int ZipCode { get; set; }
}
{% endhighlight %}

So the DTO is an exact duplicate from the entity in this case.

Lets create the Mapper next (which is really obvious):

### The mapper

{% highlight cs %}
public interface IHouseMapper
    {
        HouseDto MapToDto(HouseEntity houseEntity);
        HouseEntity MapToEntity(HouseDto houseDto);
}
{% endhighlight %}

{% highlight cs %}
public class HouseMapper : IHouseMapper
    {
        public HouseDto MapToDto(HouseEntity houseEntity)
        {
            return new HouseDto()
            {
                Id = houseEntity.Id,
                ZipCode = houseEntity.ZipCode,
                City = houseEntity.City,
                Street = houseEntity.Street
            };
        }

        public HouseEntity MapToEntity(HouseDto houseDto)
        {
            return new HouseEntity()
            {
                Id = houseDto.Id,
                ZipCode = houseDto.ZipCode,
                City = houseDto.City,
                Street = houseDto.Street
            };
        }
}
{% endhighlight %}

So here we are just mapping from one to another. Simple case. This can get more complex but it should do it for this time.

After we created the mapper we want to have this mapper instanciated **every time a request comes in**. For this we use the build in DI-container in ASP.NET Core.

So go to Startup.cs and add the line

```services.AddTransient<IHouseMapper, HouseMapper>();```

in the "ConfigureServices"-Method. It should look like this then:

{% highlight cs %}
public void ConfigureServices(IServiceCollection services)
{
            services.AddTransient<IHouseMapper, HouseMapper>();
            // Add framework services.
            services.AddMvc();
}
{% endhighlight %}


### The Repository

The repository is responsible for providing and saving the data. Normally you would use da database here.

Like:

[https://github.com/FabianGosebrink/ASPNET-Core-Entity-Framework-6](https://github.com/FabianGosebrink/ASPNET-Core-Entity-Framework-6)

[https://github.com/FabianGosebrink/ASPNET-Core-Entity-Framework-Core](https://github.com/FabianGosebrink/ASPNET-Core-Entity-Framework-Core)

But for this time we will use like a static list where objects are added and removed.

{% highlight cs %}
public interface IHouseRepository
    {
        List<HouseEntity> GetAll();
        HouseEntity GetSingle(int id);
        HouseEntity Add(HouseEntity toAdd);
        HouseEntity Update(HouseEntity toUpdate);
        void Delete(int id);
}
{% endhighlight %}

{% highlight cs %}
public class HouseRepository : IHouseRepository
    {
        readonly Dictionary<int, HouseEntity> _houses = new Dictionary<int, HouseEntity>();

        public HouseRepository()
        {
            _houses.Add(1, new HouseEntity() { City = "Town1", Id = 1, Street = "Street1", ZipCode = 1234 });
            _houses.Add(2, new HouseEntity() { City = "Town2", Id = 2, Street = "Street2", ZipCode = 1234 });
            _houses.Add(3, new HouseEntity() { City = "Town3", Id = 3, Street = "Street3", ZipCode = 1234 });
            _houses.Add(4, new HouseEntity() { City = "Town4", Id = 4, Street = "Street4", ZipCode = 1234 });
        }

        public List<HouseEntity> GetAll()
        {
            return _houses.Select(x => x.Value).ToList();
        }

        public HouseEntity GetSingle(int id)
        {
            return _houses.FirstOrDefault(x => x.Key == id).Value;
        }

        public HouseEntity Add(HouseEntity toAdd)
        {
            int newId = !GetAll().Any() ? 1 : GetAll().Max(x => x.Id) + 1;
            toAdd.Id = newId;
            _houses.Add(newId, toAdd);
            return toAdd;
        }

        public HouseEntity Update(HouseEntity toUpdate)
        {
            HouseEntity single = GetSingle(toUpdate.Id);

            if (single == null)
            {
                return null;
            }

            _houses[single.Id] = toUpdate;
            return toUpdate;
        }

        public void Delete(int id)
        {
            _houses.Remove(id);
        }
}
{% endhighlight %}

A normal interface using Entity Framwork could look like this btw:

{% highlight cs %}
public interface IExampleRepository
    {
        IEnumerable<MyModel> GetAll();
        MyModel GetSingle(int id);
        MyModel Add(MyModel toAdd);
        MyModel Update(MyModel toUpdate);
        void Delete(MyModel toDelete);
        int Save();
}
{% endhighlight %}

taken from [https://github.com/FabianGosebrink/ASPNET-Core-Entity-Framework-Core/blob/master/src/AspnetCoreEFCoreExample/Repositories/IExampleRepository.cs](https://github.com/FabianGosebrink/ASPNET-Core-Entity-Framework-Core/blob/master/src/AspnetCoreEFCoreExample/Repositories/IExampleRepository.cs)

See the "Save()"-Method here. But for this time it should be good with the list. We are not focussing on the Database here.

So we do have the repository to save the data. Let's make it available through DI in the Startup. This time we do NOT want to have a new repo every request, so we will add a singleton this time.

In a normal case with a real DB you would not do that

Example for EF 6 is here

[https://github.com/FabianGosebrink/ASPNET-Core-Entity-Framework-6/blob/master/src/AspnetCoreEF6Example/Startup.cs#L32](https://github.com/FabianGosebrink/ASPNET-Core-Entity-Framework-6/blob/master/src/AspnetCoreEF6Example/Startup.cs#L32)

[https://github.com/FabianGosebrink/ASPNET-Core-Entity-Framework-Core/blob/master/src/AspnetCoreEFCoreExample/Startup.cs#L32](https://github.com/FabianGosebrink/ASPNET-Core-Entity-Framework-Core/blob/master/src/AspnetCoreEFCoreExample/Startup.cs#L32)
    
    
But this time we will use a singleton. Shame on me so far.

```services.AddSingleton<IHouseRepository, HouseRepository>();```

So the whole Startup.cs is now like:

{% highlight cs %}
public void ConfigureServices(IServiceCollection services)
{
    services.AddSingleton<IHouseRepository, HouseRepository>();

    services.AddTransient<IHouseMapper, HouseMapper>();
    // Add framework services.
    services.AddMvc();
}
{% endhighlight %}

With this we can start using the whole construct in a controller like this:

{% highlight cs %}
[Route("api/[controller]")]
public class HouseController : Controller
{
    private readonly IHouseMapper _houseMapper;
    private readonly IHouseRepository _houseRepository;

    public HouseController(IHouseMapper houseMapper, IHouseRepository houseRepository)
    {
        _houseMapper = houseMapper;
        _houseRepository = houseRepository;
    }

    [HttpGet]
    public IActionResult Get()
    {
        try
        {
            return Ok(_houseRepository.GetAll().Select(x => _houseMapper.MapToDto(x)));
        }
        catch (Exception exception)
        {
            //logg exception or do anything with it
            return StatusCode((int) HttpStatusCode.InternalServerError);
        }
    }

    [HttpGet("{id:int}", Name = "GetSingleHouse")]
    public IActionResult GetSingle(int id)
    {
        try
        {
            HouseEntity houseEntity = _houseRepository.GetSingle(id);

            if (houseEntity == null)
            {
                return NotFound();
            }

            return Ok(_houseMapper.MapToDto(houseEntity));
        }
        catch (Exception exception)
        {
            //logg exception or do anything with it
            return StatusCode((int)HttpStatusCode.InternalServerError);
        }
    }

    [HttpPatch("{id:int}")]
    public IActionResult Patch(int id, [FromBody] JsonPatchDocument<HouseDto> housePatchDocument)
    {
        try
        {
            if (housePatchDocument == null)
            {
                return BadRequest();
            }

            if (!ModelState.IsValid)
            {
                return BadRequest(ModelState);
            }

            HouseEntity houseEntity = _houseRepository.GetSingle(id);

            if (houseEntity == null)
            {
                return NotFound();
            }

            HouseDto existingHouse = _houseMapper.MapToDto(houseEntity);

            housePatchDocument.ApplyTo(existingHouse, ModelState);

            if (!ModelState.IsValid)
            {
                return BadRequest(ModelState);
            }

            _houseRepository.Update(_houseMapper.MapToEntity(existingHouse));

            return Ok(existingHouse);
        }
        catch (Exception exception)
        {
            //logg exception or do anything with it
            return StatusCode((int)HttpStatusCode.InternalServerError);
        }
    }

    [HttpPost]
    public IActionResult Create([FromBody] HouseDto houseDto)
    {
        try
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

            _houseRepository.Add(houseEntity);

            return CreatedAtRoute("GetSingleHouse", new { id = houseEntity.Id }, _houseMapper.MapToDto(houseEntity));
        }
        catch (Exception exception)
        {
            //logg exception or do anything with it
            return StatusCode((int)HttpStatusCode.InternalServerError);
        }
    }

    [HttpPut("{id:int}")]
    public IActionResult Update(int id, [FromBody] HouseDto houseDto)
    {
        try
        {
            if (houseDto == null)
            {
                return BadRequest();
            }

            if (!ModelState.IsValid)
            {
                return BadRequest(ModelState);
            }

            HouseEntity houseEntityToUpdate = _houseRepository.GetSingle(id);

            if (houseEntityToUpdate == null)
            {
                return NotFound();
            }

            houseEntityToUpdate.ZipCode = houseDto.ZipCode;
            houseEntityToUpdate.Street = houseDto.Street;
            houseEntityToUpdate.City = houseDto.City;

            _houseRepository.Update(houseEntityToUpdate);

            return Ok(_houseMapper.MapToDto(houseEntityToUpdate));
        }
        catch (Exception exception)
        {
            //logg exception or do anything with it
            return StatusCode((int)HttpStatusCode.InternalServerError);
        }
    }

    [HttpDelete("{id:int}")]
    public IActionResult Delete(int id)
    {
        try
        {
            HouseEntity houseEntityToDelete = _houseRepository.GetSingle(id);

            if (houseEntityToDelete == null)
            {
                return NotFound();
            }

            _houseRepository.Delete(id);

            return NoContent();
        }
        catch (Exception exception)
        {
            //logg exception or do anything with it
            return StatusCode((int)HttpStatusCode.InternalServerError);
        }
    }
}
{% endhighlight %}

Now we can go ahead and test this with a tool like postman or whatever:

![Create an ASP.NET Core 1.0 WebAPI]({{site.baseurl}}assets/articles/wp-content/uploads/2016/07/AspNetCoreWebApi1.jpg)

![AspNetCoreWebApi3]({{site.baseurl}}assets/articles/wp-content/uploads/2016/07/AspNetCoreWebApi3.jpg)
