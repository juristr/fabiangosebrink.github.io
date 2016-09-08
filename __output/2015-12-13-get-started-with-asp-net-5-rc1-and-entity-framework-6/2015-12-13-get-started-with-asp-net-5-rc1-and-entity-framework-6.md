---
id: 869
title: Get started with ASP.NET Core 1.0 and Entity Framework 6
date: 2015-12-13T20:20:21+00:00
author: Fabian Gosebrink
layout: post
tags: asp.netmvc5 automapper codefirst databasecontext entityframework json project.json webapi 
logo: 'assets/images/logo_small.png'
navigation: True
cover: 'assets/images/download_edit_dark.jpg'
subclass: 'post tag-speeches'
---

Hey,

today I want to show you how to get started with ASP.NET 5 and Entity Framework 6.

If you start with the new templates for ASP.NET 5 you will notice in a short time that examples are going the EF 7 way. But if you want to stay at Entity Framework 6 as long as 7 is not in a final release or just to move from an older version step by step you can follow this instructions here. In this blog post I want to show you how to include a database with a connectionstring saved in a json file with the new ASP.NET 5 RC1.

> Note: At the time of this post ASP.NET was at RC1 status. There might be changes until its completely an final released. However: If you want to dive into new functionalities: Keep reading.

## Get started with ASP.NET 5 RC1 and Entity Framework 6 :

First of all you need to start an new project with the new RC1 of ASP.NET like this:

&nbsp;

<a href="{{site.baseurl}}assets/images/blogs/2015-12/6e269185-2135-4f67-9463-ef5c2fc2d831.jpg" rel="attachment wp-att-872"><img class="aligncenter wp-image-872" src="{{site.baseurl}}assets/images/blogs/2015-12/6e269185-2135-4f67-9463-ef5c2fc2d831.jpg" alt="Ef6Example" width="639" height="498" srcset="{{site.baseurl}}assets/images/blogs/2015-12/6e269185-2135-4f67-9463-ef5c2fc2d831.jpg 786w, http://offering.solutions/wp-content/uploads/2015/12/Ef6Example-300x234.jpg 300w, http://offering.solutions/wp-content/uploads/2015/12/Ef6Example-768x599.jpg 768w" sizes="(max-width: 639px) 100vw, 639px" /></a>

&nbsp;

This will create you a new nearly empty solution following the new standards with all configs in *.json files and so on.

> This example is only made fot the full version of the .net-Framework. So the core version will not be supported with this example.

The first step we a re going to do is adding the dependency of the Entity Framework to our solution via the project.json file. For this only put the line

<pre class="lang:js decode:true ">"EntityFramework": "6.1.3"</pre>

at the end of you dependencies section like this:

<a href="{{site.baseurl}}assets/images/blogs/2015-12/e207c26b-8bc0-4746-aeab-3776b98405e2.jpg" rel="attachment wp-att-873"><img class="aligncenter size-full wp-image-873" src="{{site.baseurl}}assets/images/blogs/2015-12/e207c26b-8bc0-4746-aeab-3776b98405e2.jpg" alt="Ef6Example_02" width="623" height="222" srcset="{{site.baseurl}}assets/images/blogs/2015-12/e207c26b-8bc0-4746-aeab-3776b98405e2.jpg 623w, http://offering.solutions/wp-content/uploads/2015/12/Ef6Example_02-300x107.jpg 300w" sizes="(max-width: 623px) 100vw, 623px" /></a>This will get Visual Studio 2015 to update your dependencies including the Entity Framework.

Now you can create a new class named like your Context. in this case this will be &#8220;MyEf6EntityFrameworkContext&#8221;.

<a href="{{site.baseurl}}assets/images/blogs/2015-12/ca264317-b4e3-4e88-bb98-f2d401fa06c1.jpg" rel="attachment wp-att-874"><img class="aligncenter size-full wp-image-874" src="{{site.baseurl}}assets/images/blogs/2015-12/ca264317-b4e3-4e88-bb98-f2d401fa06c1.jpg" alt="Ef6Example_03" width="548" height="236" srcset="{{site.baseurl}}assets/images/blogs/2015-12/ca264317-b4e3-4e88-bb98-f2d401fa06c1.jpg 548w, http://offering.solutions/wp-content/uploads/2015/12/Ef6Example_03-300x129.jpg 300w" sizes="(max-width: 548px) 100vw, 548px" /></a>Be sure to use the &#8220;base&#8221;-functionality, because we will need it when passing the connectionstring to the context reading it out of the *.json file.

Back in our Startup.cs-File we are including a file called &#8220;appsettings.json&#8221;. Lets go and add our Connectionstring to this file:

<a href="{{site.baseurl}}assets/images/blogs/2015-12/bb036de1-b860-42d5-8d1c-c646453b5be9.jpg" rel="attachment wp-att-875"><img class="aligncenter size-full wp-image-875" src="{{site.baseurl}}assets/images/blogs/2015-12/bb036de1-b860-42d5-8d1c-c646453b5be9.jpg" alt="Ef6Example_04" width="1056" height="229" srcset="{{site.baseurl}}assets/images/blogs/2015-12/bb036de1-b860-42d5-8d1c-c646453b5be9.jpg 1056w, http://offering.solutions/wp-content/uploads/2015/12/Ef6Example_04-300x65.jpg 300w, http://offering.solutions/wp-content/uploads/2015/12/Ef6Example_04-768x167.jpg 768w, http://offering.solutions/wp-content/uploads/2015/12/Ef6Example_04-1024x222.jpg 1024w" sizes="(max-width: 1056px) 100vw, 1056px" /></a>This should look quite familiar because of the connectionstring you knew from the web.config in the previous asp.net-versions.

## What we did so far:

At this point we added the connectionstring to the config file we will consume in the startup.cs and we created a databasecontext like we know it which will provide us any data in the future.

## Go ahead!

Next thing we have to do is getting the config and our databasecontext married 🙂

This can be done putting a single line in our &#8220;Startup.cs&#8221;-File.

Just add

<pre class="lang:c# decode:true ">services.AddScoped&lt;MyEf6EntityFrameworkContext&gt;((s) =&gt; new MyEf6EntityFrameworkContext(Configuration["Data:Ef6ExampleConnectionString"]));</pre>

in the method &#8220;ConfigureServices&#8221; in your Startup.cs.

<pre class="lang:c# decode:true">public void ConfigureServices(IServiceCollection services)
        {
            // Add framework services.
            services.AddMvc();
            services.AddScoped&lt;MyEf6EntityFrameworkContext&gt;((s) =&gt; new MyEf6EntityFrameworkContext(Configuration["Data:Ef6ExampleConnectionString"]));
        }</pre>

This line will read the configuration and get the connection-string out of it and use it for establishing the connection. pay attention to the scoped adding. This is because the context should be generated for every single request. You can grab an overview of all lifestyles here, even it&#8217;s a bit outdated: [Dependency Injection in ASP.NET vNext](http://blogs.msdn.com/b/webdev/archive/2014/06/17/dependency-injection-in-asp-net-vnext.aspx)

## The Model

&nbsp;

You can now go ahead and install my [Unit of Work](https://github.com/FabianGosebrink/OfferingSolutions-UnitOfWork) via nuget and create a repository like this:

<pre class="lang:c# decode:true ">public interface IExampleRepository : IRepositoryContext&lt;MyModel&gt;
    {
    }</pre>

<pre class="lang:c# decode:true">public class ExampleRepository : RepositoryContextImpl&lt;MyModel&gt;, IExampleRepository
    {
        public ExampleRepository(MyEf6EntityFrameworkContext databaseContext) : base(databaseContext)
        {
        }
    }</pre>

With a model like this (for example):

<pre class="lang:c# decode:true " title="MyModel.cs">public class MyModel
    {
        [Key]
        public int Id { get; set; }
        public string Name { get; set; }
    }</pre>

Meanwhile I also added [Automappers](https://github.com/AutoMapper/AutoMapper/wiki/Getting-started) and a ViewModel to map between those two:

<pre class="lang:c# decode:true" title="MyModelViewModel.cs">public class MyModelViewModel
    {
        public int Id { get; set; }
        [Required]
        public string Name { get; set; }
    }</pre>

&nbsp;

Now we have to bring it to the build in DI in ASP.NET:

<pre class="lang:c# decode:true ">public void ConfigureServices(IServiceCollection services)
        {
            // Add framework services.
            services.AddMvc();
            services.AddScoped&lt;IExampleRepository, ExampleRepository&gt;();
            services.AddScoped&lt;MyEf6EntityFrameworkContext&gt;((s) =&gt; new MyEf6EntityFrameworkContext(Configuration["Data:Ef6ExampleConnectionString"]));
        }</pre>

and the automapping:

<pre class="lang:c# decode:true ">public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
        {
            Mapper.Initialize(config =&gt;
            {
                config.CreateMap&lt;MyModel, MyModelViewModel&gt;().ReverseMap();
            });

            //...
        }</pre>

&nbsp;

In the end you only have to build up a controller which gives and takes the values as your API:

<pre class="lang:c# decode:true " title="MyModelController">[Route("api/[controller]")]
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
                List&lt;MyModel&gt; MyModels = _exampleRepository.GetAll().ToList();
                return new JsonResult(MyModels.Select(x =&gt; Mapper.Map&lt;MyModelViewModel&gt;(x)));
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

                return new HttpOkObjectResult(Mapper.Map&lt;MyModelViewModel&gt;(MyModel));
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

                MyModel item = Mapper.Map&lt;MyModel&gt;(viewModel);

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

                if (save &gt; 0)
                {
                    return new HttpOkObjectResult(Mapper.Map&lt;MyModelViewModel&gt;(singleById));
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

                if (save &gt; 0)
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
    }</pre>

Thats it. If you now going to use the DatabaseContext it will create the database for you with the new ASP.NET 5 RC1.

<a href="{{site.baseurl}}assets/images/blogs/2015-12/0d72ebea-fc04-411f-b6b9-84b55a90e975.jpg" rel="attachment wp-att-889"><img class="aligncenter size-full wp-image-889" src="{{site.baseurl}}assets/images/blogs/2015-12/0d72ebea-fc04-411f-b6b9-84b55a90e975.jpg" alt="Ef6Example_05" width="228" height="240" /></a>

You can now go ahead and add an item with e.g. postman:

<a href="{{site.baseurl}}assets/images/blogs/2015-12/98e13d7f-f2fd-4cec-b3d6-b2034941ad97.jpg" rel="attachment wp-att-890"><img class="aligncenter wp-image-890" src="{{site.baseurl}}assets/images/blogs/2015-12/98e13d7f-f2fd-4cec-b3d6-b2034941ad97.jpg" alt="Ef6Example_06" width="1094" height="648" srcset="{{site.baseurl}}assets/images/blogs/2015-12/98e13d7f-f2fd-4cec-b3d6-b2034941ad97.jpg 1415w, http://offering.solutions/wp-content/uploads/2015/12/Ef6Example_06-300x178.jpg 300w, http://offering.solutions/wp-content/uploads/2015/12/Ef6Example_06-768x455.jpg 768w, http://offering.solutions/wp-content/uploads/2015/12/Ef6Example_06-1024x606.jpg 1024w" sizes="(max-width: 1094px) 100vw, 1094px" /></a>and it will be stored in the database:

<a href="{{site.baseurl}}assets/images/blogs/2015-12/b2445f76-567f-435d-a7d4-023a9c71e66a.jpg" rel="attachment wp-att-891"><img class="aligncenter size-full wp-image-891" src="{{site.baseurl}}assets/images/blogs/2015-12/b2445f76-567f-435d-a7d4-023a9c71e66a.jpg" alt="Ef6Example_07" width="646" height="388" srcset="{{site.baseurl}}assets/images/blogs/2015-12/b2445f76-567f-435d-a7d4-023a9c71e66a.jpg 646w, http://offering.solutions/wp-content/uploads/2015/12/Ef6Example_07-300x180.jpg 300w" sizes="(max-width: 646px) 100vw, 646px" /></a> This example is also available on [Github](https://github.com/FabianGosebrink/Asp.Net5WithEntityFramework6)

&nbsp;

Hope this helps

Regards

Fabian