---
id: 939
title: 'Start with VS Code, Angular2, Typescript &#038; ASP.NET Core 1.0'
date: 2016-01-25T19:47:01+00:00
author: Fabian Gosebrink
layout: post
tags: angular2 asp.netcore visualstudio visualstudiocode 
logo: 'assets/images/logo_small.png'
navigation: True
cover: 'assets/images/download_edit_dark.jpg'
subclass: 'post tag-speeches'
disqus: true
categories: articles
---

Hey,

with this bIog I want to show how to start with Visual Studio (Code), Angular2 beta 0, Typescript & ASP.NET Core 1.0. Find the GitHub-Repo below.

Nowadays you hear a lot of new technologies around the web stuff and with this post I want to show you a get-starting-guide how to start with the following technologies in general. I want to show how to set up an environment where you can start programming ASP.NET and AngularJs 2.0. As I started I spent a lot of time setting up all my tools and stuff. If you want to save this time: Go ahead reading.

So I want to have an environment where I can build an Angular 2 web application with typescript, having an api where I can call to build with ASP.NET Core 1.0. We will also use [Visual Studio Code](https://code.visualstudio.com/) to get the client-side programming going.

## Lets start:

First thing to do is setting up our Visual Studio. Create a new application and choose the ASP.NET 5 Templates. I guess they will be renamed later 😉

![Alttext]({{site.baseurl}}assets/articles/2016-01-25/50f21617-631c-482b-9710-63b9b80944a5.png)

![Alttext]({{site.baseurl}}assets/articles/2016-01-25/af42a271-d958-42fa-875c-cef340fecdda.png)

You will get an empty solution with no dependencies so far. Lets go and add some: open up the "project.json" and add the dependencies like this:

<pre class="lang:js decode:true">"dependencies": {
        "Microsoft.AspNet.IISPlatformHandler": "1.0.0-rc1-final",
        "Microsoft.AspNet.Server.Kestrel": "1.0.0-rc1-final",
        "Microsoft.AspNet.Mvc": "6.0.0-rc1-final",
        "Microsoft.AspNet.StaticFiles": "1.0.0-rc1-final",
        "Microsoft.Extensions.Logging.Console": "1.0.0-rc1-final",
        "Microsoft.Extensions.Logging.Debug": "1.0.0-rc1-final",
        "AutoMapper": "4.1.1"
    },</pre>

What we are adding is MVC, because we will need it for setting up the API, some logging things and AutoMappers to map between our DTOs and our real Entities. We wont use it in this tutorial, but for a kickstart its good to have them.

Visual Studio should now go ahead and resolve, download and install these packages.

![Alttext]({{site.baseurl}}assets/articles/2016-01-25/9e2509d5-ec6d-4835-9d94-8aa8d350515b.png)

Now go on and add an "app"-Folder, a "js"-folder and a "css"-folder to the wwwroot-folder.

> Everything in the wwwroot-folder is going to be served to the client. This is where our app lives. You can also build your application outside this folder and copy the relevant files in it via gulp etc. I am not doing this in this case because its easier to debug in the when launching. If you have a better approach I would love to hear it 🙂

![Alttext]({{site.baseurl}}assets/articles/2016-01-25/87964802-3a14-48e1-b9e4-40e023717e40.png)

To get started with our web application we need an index.html. Lets add it and put nothing in it so far.

![Alttext]({{site.baseurl}}assets/articles/2016-01-25/d0dcd9be-e3f5-4899-ba50-be0f29372de8.png)

## The Serverside/API

Lets get started with the serverside: To wire up everything we need to configure our application in the Startup.cs.

<pre class="lang:c# decode:true">public class Startup
{
	public Startup(IHostingEnvironment env)
	{
		var builder = new ConfigurationBuilder()
			.AddJsonFile("appsettings.json")
			.AddEnvironmentVariables();
		Configuration = builder.Build();
	}

	public IConfigurationRoot Configuration { get; set; }

	public void ConfigureServices(IServiceCollection services)
	{
		services.AddMvc();
	}

	// This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
	public void Configure(IApplicationBuilder app, ILoggerFactory loggerFactory)
	{
		loggerFactory.AddConsole(Configuration.GetSection("Logging"));
		loggerFactory.AddDebug();
		
		app.UseIISPlatformHandler();

		app.UseStaticFiles();

		app.UseMvc();
	}

	public static void Main(string[] args) =&gt; WebApplication.Run&lt;Startup&gt;(args);
}</pre>

Because at the beginning of this page we are consuming an "appsettings.json" we should also create it. Create a new file called like this beside the Startup.cs and paste some logging information to it. You can add some informations you want to have in there later if you want. For the start, this should be okay.

<pre class="lang:js decode:true ">{
  "Logging": {
    "IncludeScopes": false,
    "LogLevel": {
      "Default": "Verbose",
      "System": "Information",
      "Microsoft": "Information"
    }
  }
}
</pre>

Our application should now be able to start up. Just go to the commandline and type "dnx web" in it. This will start a webserver and serve all files in wwwroot.

![Alttext]({{site.baseurl}}assets/articles/2016-01-25/5872d413-ac2b-4561-9598-7c4b537a3af5.png)

![Alttext]({{site.baseurl}}assets/articles/2016-01-25/7e6f8b76-7cb3-41e6-b220-fc8a2603a4f5.png)

Great so far! Now we can add a controller!

For this go ahead in the root of the project and add a folder "Controller" which wil hold all the controllers for us. Then you can add an standardcontroller for the first run.

> I like to stay to IActionResult-Interface in my controllers. It just gives me a way I am used to and is clearer for me. You do not have to do this. The controller is just for demo purposes.

![Alttext]({{site.baseurl}}assets/articles/2016-01-25/eef224b8-ae35-45fa-94fd-7b57f036fd0c.png)
If we now run the server again with "dnx web" it will start and via Postman we can check if everything works correctly:

![Alttext]({{site.baseurl}}assets/articles/2016-01-25/4e0f0af8-1c9c-4551-ab7a-fe07446138f0.png)

## The Client

Great. So lets start clientside. What we need is an AngularJs 2.0 environment. For the tooling I made the best experiences using [Visual Studio Code](https://code.visualstudio.com/).

We already defined the wwwroot-Folder which holds every file which is transported to the client.

![Alttext]({{site.baseurl}}assets/articles/2016-01-25/748ca475-801b-45f3-ab14-525d0381e2f9.png)

As we start developing AngularJs 2.0 we have to prepare our environment for working with Typescript. I already have an instruction how to start in a previous Blogpost [Getting started with Visual Studio Code & Typescript](http://offering.solutions/subclass: 'post tag-speeches'
disqus: true
categories: articles
disqus: true
categories: articles/asp-net/getting-started-with-visual-studio-code-and-typescript/). But lets cover some thing in short to get it customised for this AngularJs 2.0 case.

Lets add a file for configuring the typescript-options for our project: tsconfig.json to the root of our project.

<pre class="lang:js decode:true ">{
    "compilerOptions": {
        "target": "es5",
        "module": "commonjs",
        "removeComments": true,
        "emitDecoratorMetadata": true,
        "experimentalDecorators": true,
        "noEmitHelpers": false,
        "sourceMap": true
    },
    "exclude": [
        "node_modules"
    ],
    "compileOnSave": false,
    "buildOnSave": false
}</pre>

### Adding Angular2

Normally you would add AngularJs 2 via the tsd-command. But in this case AngularJs 2 is served via npm. To get Angular2 we have to add a package.json to the root of our project.

<pre class="lang:c# decode:true">{
    "name": "AspNet5Angular2Demo",
    "version": "1.0.0",
    "description": "",
    "main": "wwwroot/index.html",
    "author": "",
    "license": "ISC",
    "dependencies": {
        "angular2": "2.0.0-beta.0",
        "systemjs": "0.19.6",
        "es6-promise": "^3.0.2",
        "es6-shim": "^0.33.3",
        "reflect-metadata": "0.1.2",
        "rxjs": "5.0.0-beta.0",
        "zone.js": "0.5.10"
    },
    "devDependencies": {
        "typescript": "^1.7.3"
    }
}</pre>

or just type "npm init" to answer all questions and get an project.json file generated.

After this you can install AngularJs 2 and all other dependencies via "npm install" or visual Studio will do this for you automatically. Just just have to wait a few seconds. It will add a node_modules folder to your root application and copy all files, also the *.d.ts-files you need to develop with typescript.

Last thing to do is copying all your needed files into the "js"-folder you created on clientside.

![Alttext]({{site.baseurl}}assets/articles/2016-01-25/99c6eae8-059e-428f-b715-bdbd5bb62ccb.png)
(Ignore the jquery and signalr files I had addinitonally in the example, its for the next blogpost 😉 )

Now go ahead in you index.html and link to the files in the client js folder:

<pre class="lang:xhtml decode:true ">&lt;html&gt;

  &lt;head&gt;
    &lt;title&gt;ASPNET5Angular2Demo&lt;/title&gt;
  &lt;/head&gt;

  &lt;!-- 3. Display the application --&gt;
  &lt;body&gt;
    &lt;my-app&gt;Loading...&lt;/my-app&gt;
	
	  &lt;!-- 1. Load libraries --&gt;
    &lt;!-- IE required polyfills, in this exact order --&gt;
    &lt;script src="js/es6-shim.min.js"&gt;&lt;/script&gt;
    &lt;script src="js/system-polyfills.js"&gt;&lt;/script&gt;

    &lt;script src="js/angular2-polyfills.js"&gt;&lt;/script&gt;
    &lt;script src="js/system.src.js"&gt;&lt;/script&gt;
    &lt;script src="js/Rx.js"&gt;&lt;/script&gt;
    &lt;script src="js/angular2.dev.js"&gt;&lt;/script&gt;

    &lt;!-- 2. Configure SystemJS --&gt;
    &lt;script&gt;
      System.config({
        packages: {        
          app: {
            defaultExtension: 'js'
          }
        }
      });
      System.import('app/boot')
            .then(null, console.error.bind(console));
    &lt;/script&gt;
  &lt;/body&gt;

&lt;/html&gt;</pre>

## Finally&#8230;

We are done so far:

You can now go ahead and develop you application in Typescript and Angular2. You could start with the [AngularJs 2](https://angular.io/docs/ts/latest/quickstart.html) quickstart. All files you need should be there. Its important to add the boot.ts and application.ts to get an entry point and bootstrapping your application.

> If you, like me, always forget to compile the typescript files: open a cmd in the app folder and just type "tsc -w". This activates a typescript watcher using the tsconfig in the root of the project and will always compile the \*.ts files into \*.js files if there are some changes

### Further steps:

You could now start using gulp/grunt to copy files to the locations automatically instead of doing it manually. Or you could start consuming the api using the http-Service like shown [here](https://auth0.com/blog/2015/10/15/angular-2-series-part-3-using-http/) or wait for one of my next blog-posts 😉

[Angular 2 Beta 0 Starter Template on GitHub](https://github.com/FabianGosebrink/Angular2Beta0StarterTemplate)

Regards & HTH

Fabian
