---
title: Get started with ASP.NET Core and Entity Framework 6
date: 2015-12-13 20:20
author: Fabian Gosebrink
layout: post
tags: ASP.NET MVC Automapper Code First Database Context Entity Framework JSON project.json WebAPI
logo: 'assets/images/logo_small.png'
navigation: True
cover: 'assets/images/aerial-view-of-laptop-notebook-mobile-phone-and-coffee-cup-on-wooden-table.jpg'
subclass: 'post tag-speeches'
disqus: true
categories: articles
---

Hey,

today I want to show you how to get started with ASP.NET 5 and Entity Framework 6.

If you start with the new templates for ASP.NET 5 you will notice in a short time that examples are going the EF 7 way. But if you want to stay at Entity Framework 6 as long as 7 is not in a final release or just to move from an older version step by step you can follow this instructions here. In this blog post I want to show you how to include a database with a connectionstring saved in a json file with the new ASP.NET Core.

>Note: At the time of this post ASP.NET was at RC1 status. There might be changes until its completely an final released. However: If you want to dive into new functionalities: Keep reading.

## Get started with ASP.NET Core and Entity Framework 6 :

First of all you need to start an new project with the new ASP.NET like this:

![Ef6Example]({{site.baseurl}}assets/articles/wp-content/uploads/2015/12/Ef6Example.jpg)

This will create you a new nearly empty solution following the new standards with all configs in *.json files and so on.

>This example is only made fot the full version of the .net-Framework. So the core version will not be supported with this example.

The first step we a re going to do is adding the dependency of the Entity Framework to our solution via the project.json file. For this only put the line

```"EntityFramework": "6.1.3"```

at the end of you dependencies section like this:

![Ef6Example_02]({{site.baseurl}}assets/articles/wp-content/uploads/2015/12/Ef6Example_02.jpg)

This will get Visual Studio 2015 to update your dependencies including the Entity Framework.

Now you can create a new class named like your Context. in this case this will be "MyEf6EntityFrameworkContext".

![Ef6Example_03]({{site.baseurl}}assets/articles/wp-content/uploads/2015/12/Ef6Example_03.jpg)

Be sure to use the "base"-functionality, because we will need it when passing the connectionstring to the context reading it out of the *.json file.

Back in our Startup.cs-File we are including a file called "appsettings.json". Lets go and add our Connectionstring to this file:

![Ef6Example_04]({{site.baseurl}}assets/articles/wp-content/uploads/2015/12/Ef6Example_04.jpg)

This should look quite familiar because of the connectionstring you knew from the web.config in the previous asp.net-versions.

## What we did so far:

At this point we added the connectionstring to the config file we will consume in the startup.cs and we created a databasecontext like we know it which will provide us any data in the future.

## Go ahead!

Next thing we have to do is getting the config and our databasecontext married :-)

This can be done putting a single line in our "Startup.cs"-File.

Just add

{% highlight cs %}
services.AddScoped<MyEf6EntityFrameworkContext>((s) => new MyEf6EntityFrameworkContext(Configuration["Data:Ef6ExampleConnectionString"]));
{% endhighlight %}

in the method "ConfigureServices" in your Startup.cs.

{% highlight cs %}
public void ConfigureServices(IServiceCollection services)
{
    // Add framework services.
    services.AddMvc();
    services.AddScoped<MyEf6EntityFrameworkContext>((s) => new MyEf6EntityFrameworkContext(Configuration["Data:Ef6ExampleConnectionString"]));
}
{% endhighlight %}

This line will read the configuration and get the connection-string out of it and use it for establishing the connection. pay attention to the scoped adding. This is because the context should be generated for every single request. You can grab an overview of all lifestyles here, even it's a bit outdated: [Dependency Injection in ASP.NET vNext](http://blogs.msdn.com/b/webdev/archive/2014/06/17/dependency-injection-in-asp-net-vnext.aspx)

## The Model

You can now go ahead and install my [Unit of Work](https://github.com/FabianGosebrink/OfferingSolutions-UnitOfWork) via nuget and create a repository like this:

{% highlight cs %}
public interface IExampleRepository : IRepositoryContext<MyModel>
    {
    }
{% endhighlight %}

{% highlight cs %}
public class ExampleRepository : RepositoryContextImpl<MyModel>, IExampleRepository
{
    public ExampleRepository(MyEf6EntityFrameworkContext databaseContext) : base(databaseContext)
    {
    }
}
{% endhighlight %}

With a model like this (for example):

{% highlight cs %}
public class MyModel
    {
        [Key]
        public int Id { get; set; }
        public string Name { get; set; }
    }
{% endhighlight %}

Meanwhile I also added [Automappers](https://github.com/AutoMapper/AutoMapper/wiki/Getting-started) and a ViewModel to map between those two:

{% highlight cs %}
public class MyModelViewModel
{
    public int Id { get; set; }
    [Required]
    public string Name { get; set; }
}
{% endhighlight %}

Now we have to bring it to the build in DI in ASP.NET:

{% highlight cs %}
public void ConfigureServices(IServiceCollection services)
{
    // Add framework services.
    services.AddMvc();
    services.AddScoped<IExampleRepository, ExampleRepository>();
    services.AddScoped<MyEf6EntityFrameworkContext>((s) => new MyEf6EntityFrameworkContext(Configuration["Data:Ef6ExampleConnectionString"]));
}
{% endhighlight %}

and the automapping:

{% highlight cs %}
public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
{
    Mapper.Initialize(config =>
    {
        config.CreateMap<MyModel, MyModelViewModel>().ReverseMap();
    });

    //...
}
{% endhighlight %}

In the end you only have to build up a controller which gives and takes the values as your API:

{% highlight cs %}
[Route("api/[controller]")]
public class MyModelController
{
    private readonly IExampleRepository _exampleRepository;

    public MyModelController(IExampleRepository exampleRepository)
    {
        _exampleRepository = exampleRepository;
    }

    // GET: api/mymodel
    [HttpGet("", Name = "GetAll")]
    public IActionResult Get()
    {
        try
        {
            List<MyModel> MyModels = _exampleRepository.GetAll().ToList();
            return new JsonResult(MyModels.Select(x => Mapper.Map<MyModelViewModel>(x)));
        }
        catch (Exception)
        {
            //Do something with the exception
            return new HttpStatusCodeResult((int)HttpStatusCode.InternalServerError);
        }
    }

    // GET api/values/5
    [HttpGet("{id}", Name = "GetSingle")]
    public IActionResult Get(int id)
    {
        try
        {
            MyModel MyModel = _exampleRepository.GetSingleById(id);

            if (MyModel == null)
            {
                return new HttpNotFoundResult();
            }

            return new HttpOkObjectResult(Mapper.Map<MyModelViewModel>(MyModel));
        }
        catch (Exception ex)
        {
            //Do something with the exception
            return new HttpStatusCodeResult((int)HttpStatusCode.InternalServerError);
        }
    }

    // POST api/values
    [HttpPost]
    public IActionResult Post([FromBody]MyModelViewModel viewModel)
    {
        try
        {
            if (viewModel == null)
            {
                return new BadRequestResult();
            }

            MyModel item = Mapper.Map<MyModel>(viewModel);

            _exampleRepository.Add(item);
            int save = _exampleRepository.Save();

            return new CreatedAtRouteResult("GetSingle", new { controller = "MyModel", id = item.Id }, item);
        }
        catch (Exception exception)
        {
            //Do something with the exception
            return new HttpStatusCodeResult((int)HttpStatusCode.InternalServerError);
        }
    }

    // PUT api/values/5
    [HttpPut("{id}")]
    public IActionResult Put(int id, [FromBody]MyModelViewModel viewModel)
    {
        try
        {
            if (viewModel == null)
            {
                return new BadRequestResult();
            }

            MyModel singleById = _exampleRepository.GetSingleById(id);

            if (singleById == null)
            {
                return new HttpNotFoundResult();
            }

            singleById.Name = viewModel.Name;

            _exampleRepository.Update(singleById);
            int save = _exampleRepository.Save();

            if (save > 0)
            {
                return new HttpOkObjectResult(Mapper.Map<MyModelViewModel>(singleById));
            }

            return new BadRequestResult();
        }
        catch (Exception exception)
        {
            //Do something with the exception
            return new HttpStatusCodeResult((int)HttpStatusCode.InternalServerError);
        }
    }

    // DELETE api/values/5
    [HttpDelete("{id}")]
    public IActionResult Delete(int id)
    {
        try
        {
            MyModel singleById = _exampleRepository.GetSingleById(id);

            if (singleById == null)
            {
                return new HttpNotFoundResult();
            }

            _exampleRepository.Delete(id);
            int save = _exampleRepository.Save();

            if (save > 0)
            {
                return new NoContentResult();
            }

            return new BadRequestResult();
        }
        catch (Exception exception)
        {
            //Do something with the exception
            return new HttpStatusCodeResult((int)HttpStatusCode.InternalServerError);
        }
    }
}
{% endhighlight %}

Thats it. If you now going to use the DatabaseContext it will create the database for you with the new ASP.NET 5 RC1.

![Ef6Example_05]({{site.baseurl}}assets/articles/wp-content/uploads/2015/12/Ef6Example_05.jpg)

You can now go ahead and add an item with e.g. postman:

![Ef6Example_06]({{site.baseurl}}assets/articles/wp-content/uploads/2015/12/Ef6Example_06.jpg)

and it will be stored in the database:

[Github](https://github.com/FabianGosebrink/Asp.Net5WithEntityFramework6)

Hope this helps

Regards

Fabian
