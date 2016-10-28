---
id: 869
title: Get started with ASP.NET Core and Entity Framework Core
date: 2015-12-13T20:20:21+00:00
author: Fabian Gosebrink
layout: post
tags: asp.netmvc5 automapper codefirst databasecontext entityframework json project.json webapi 
logo: 'assets/images/logo_small.png'
navigation: True
cover: 'assets/images/download_edit_dark.jpg'
subclass: 'post tag-speeches'
disqus: true
categories: articles
---

Hey,

today I want to show you how to get started with ASP.NET 5 and Entity Framework 6.

If you start with the new templates for ASP.NET 5 you will notice in a short time that examples are going the EF 7 way. But if you want to stay at Entity Framework 6 as long as 7 is not in a final release or just to move from an older version step by step you can follow this instructions here. In this blog post I want to show you how to include a database with a connectionstring saved in a json file with the new ASP.NET 5 RC1.

> Note: At the time of this post ASP.NET was at RC1 status. There might be changes until its completely an final released. However: If you want to dive into new functionalities: Keep reading.

## Get started with ASP.NET 5 RC1 and Entity Framework 6 :

First of all you need to start an new project with the new RC1 of ASP.NET like this:

![Get started with ASP.NET Core and Entity Framework Core]({{site.baseurl}}assets/articles/2015-12-13/6e269185-2135-4f67-9463-ef5c2fc2d831.jpg)

This will create you a new nearly empty solution following the new standards with all configs in *.json files and so on.

> This example is only made fot the full version of the .net-Framework. So the core version will not be supported with this example.

The first step we a re going to do is adding the dependency of the Entity Framework to our solution via the project.json file. For this only put the line

<pre><code class="javascript">"EntityFramework": "6.1.3"</code></pre>

at the end of you dependencies section like this:

![Get started with ASP.NET Core and Entity Framework Core]({{site.baseurl}}assets/articles/2015-12-13/e207c26b-8bc0-4746-aeab-3776b98405e2.jpg)

This will get Visual Studio 2015 to update your dependencies including the Entity Framework.

Now you can create a new class named like your Context. in this case this will be "MyEf6EntityFrameworkContext".

![Get started with ASP.NET Core and Entity Framework Core]({{site.baseurl}}assets/articles/2015-12-13/ca264317-b4e3-4e88-bb98-f2d401fa06c1.jpg)

Be sure to use the "base"-functionality, because we will need it when passing the connectionstring to the context reading it out of the *.json file.

Back in our Startup.cs-File we are including a file called "appsettings.json". Lets go and add our Connectionstring to this file:

![Get started with ASP.NET Core and Entity Framework Core]({{site.baseurl}}assets/articles/2015-12-13/bb036de1-b860-42d5-8d1c-c646453b5be9.jpg)

This should look quite familiar because of the connectionstring you knew from the web.config in the previous asp.net-versions.

## What we did so far:

At this point we added the connectionstring to the config file we will consume in the startup.cs and we created a databasecontext like we know it which will provide us any data in the future.

## Go ahead!

Next thing we have to do is getting the config and our databasecontext married 🙂

This can be done putting a single line in our "Startup.cs"-File.

Just add

<pre><code class="cs">services.AddScoped<MyEf6EntityFrameworkContext>((s) => new MyEf6EntityFrameworkContext(Configuration["Data:Ef6ExampleConnectionString"]));</code></pre>

in the method "ConfigureServices" in your Startup.cs.

<pre><code class="cs">public void ConfigureServices(IServiceCollection services)
        {
            // Add framework services.
            services.AddMvc();
            services.AddScoped<MyEf6EntityFrameworkContext>((s) => new MyEf6EntityFrameworkContext(Configuration["Data:Ef6ExampleConnectionString"]));
        }</code></pre>

This line will read the configuration and get the connection-string out of it and use it for establishing the connection. pay attention to the scoped adding. This is because the context should be generated for every single request. You can grab an overview of all lifestyles here, even it's a bit outdated: [Dependency Injection in ASP.NET vNext](http://blogs.msdn.com/b/webdev/archive/2014/06/17/dependency-injection-in-asp-net-vnext.aspx)

## The Model

You can now go ahead and install my [Unit of Work](https://github.com/FabianGosebrink/OfferingSolutions-UnitOfWork) via nuget and create a repository like this:

<pre><code class="cs">public interface IExampleRepository : IRepositoryContext<MyModel>
    {
    }</code></pre>

<pre><code class="cs">public class ExampleRepository : RepositoryContextImpl<MyModel>, IExampleRepository
    {
        public ExampleRepository(MyEf6EntityFrameworkContext databaseContext) : base(databaseContext)
        {
        }
    }</code></pre>

With a model like this (for example):

<pre><code class="cs">public class MyModel
    {
        [Key]
        public int Id { get; set; }
        public string Name { get; set; }
    }</code></pre>

Meanwhile I also added [Automappers](https://github.com/AutoMapper/AutoMapper/wiki/Getting-started) and a ViewModel to map between those two:

<pre><code class="cs">public class MyModelViewModel
    {
        public int Id { get; set; }
        [Required]
        public string Name { get; set; }
    }</code></pre>

Now we have to bring it to the build in DI in ASP.NET:

<pre><code class="cs">public void ConfigureServices(IServiceCollection services)
        {
            // Add framework services.
            services.AddMvc();
            services.AddScoped<IExampleRepository, ExampleRepository>();
            services.AddScoped<MyEf6EntityFrameworkContext>((s) => new MyEf6EntityFrameworkContext(Configuration["Data:Ef6ExampleConnectionString"]));
        }</code></pre>

and the automapping:

<pre><code class="cs">public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
        {
            Mapper.Initialize(config =>
            {
                config.CreateMap<MyModel, MyModelViewModel>().ReverseMap();
            });

            //...
        }</code></pre>

In the end you only have to build up a controller which gives and takes the values as your API:

<pre><code class="cs">[Route("api/[controller]")]
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
    }</code></pre>

Thats it. If you now going to use the DatabaseContext it will create the database for you with the new ASP.NET Core.

![Get started with ASP.NET Core and Entity Framework Core]({{site.baseurl}}assets/articles/2015-12-13/0d72ebea-fc04-411f-b6b9-84b55a90e975.jpg)

You can now go ahead and add an item with e.g. postman:

![Get started with ASP.NET Core and Entity Framework Core]({{site.baseurl}}assets/articles/2015-12-13/98e13d7f-f2fd-4cec-b3d6-b2034941ad97.jpg)

and it will be stored in the database:

![Get started with ASP.NET Core and Entity Framework Core]({{site.baseurl}}assets/articles/2015-12-13/b2445f76-567f-435d-a7d4-023a9c71e66a.jpg)

This example is also available on [Github](https://github.com/FabianGosebrink/Asp.Net5WithEntityFramework6)

Hope this helps

Regards

Fabian
