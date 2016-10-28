---
id: 869
title: Angular 2 and ASP.NET Core Starter
date: 2016-01-25T20:20:21+00:00
author: Fabian Gosebrink
layout: post
tags: Angular2, Asp.Net, ASP.NET Core, Visual Studio, Visual Studio Code
logo: 'assets/images/logo_small.png'
navigation: True
cover: 'assets/images/download_edit_dark.jpg'
subclass: 'post tag-speeches'
disqus: true
categories: articles
---

With this bIog I want to give you an Angular 2 and ASP.NET Core Starter.

Code here:

<a href="https://github.com/FabianGosebrink/ASPNET-Core-Angular2-StarterTemplate">https://github.com/FabianGosebrink/ASPNET-Core-Angular2-StarterTemplate</a>

<strong>Updated to the new Versions!!</strong>

Nowadays you hear a lot of new technologies around the web stuff and with this post I want to show you a get-starting-guide how to start with the following technologies in general. I want to show how to set up an environment where you can start programming ASP.NET and AngularJs 2.0. As I started I spent a lot of time setting up all my tools and stuff. If you want to save this time: Go ahead reading.

So I want to have an environment where I can build an Angular 2 web application with typescript, having an api where I can call to build with ASP.NET Core 1.0. We will also use <a href="https://code.visualstudio.com/">Visual Studio Code</a> to get the client-side programming going.

<h2>Lets start:</h2>

First thing to do is setting up our Visual Studio. Create a new application and choose the ASP.NET 5 Templates. I guess they will be renamed later ;-)

<h2><a href="http://offering.solutions/wp-content/uploads/2016/01/AspNetCoreAngular2Demo_1.jpg"><img class="aligncenter size-full wp-image-1595" src="http://offering.solutions/wp-content/uploads/2016/01/AspNetCoreAngular2Demo_1.jpg" alt="AspNetCoreAngular2Demo_1" width="941" height="653" /></a> <a href="http://offering.solutions/wp-content/uploads/2016/01/AspNetCoreAngular2Demo_2.jpg"><img class="aligncenter size-full wp-image-1596" src="http://offering.solutions/wp-content/uploads/2016/01/AspNetCoreAngular2Demo_2.jpg" alt="AspNetCoreAngular2Demo_2" width="786" height="613" /></a></h2>

You will get an empty solution with no dependencies so far. Lets go and add some: open up the "project.json" and add this:

<pre class="lang:js decode:true ">  {
  "dependencies": {
    "Microsoft.NETCore.App": {
      "version": "1.0.0",
      "type": "platform"
    },
    "Microsoft.AspNetCore.Server.IISIntegration": "1.0.0",
    "Microsoft.AspNetCore.Server.Kestrel": "1.0.0",
    "Microsoft.Extensions.Configuration.FileExtensions": "1.0.0",
    "Microsoft.AspNetCore.Mvc.Core": "1.0.0",
    "Microsoft.AspNetCore.StaticFiles": "1.0.0",
    "Microsoft.Extensions.Configuration.Json": "1.0.0",
    "Microsoft.AspNetCore.Mvc": "1.0.0",
    "Microsoft.Extensions.Logging.Console": "1.0.0",
    "Microsoft.Extensions.Logging.Debug": "1.0.0",
    "Microsoft.Extensions.Configuration.EnvironmentVariables": "1.0.0"
  },

  "tools": {
        "Microsoft.AspNetCore.Server.IISIntegration.Tools": "1.0.0-preview2-final"
    },

  "frameworks": {
    "netcoreapp1.0": { }
  },

  "buildOptions": {
    "emitEntryPoint": true,
    "preserveCompilationContext": true
  },

  "runtimeOptions": {
    "configProperties": {
      "System.GC.Server": true
    }
  },

  "publishOptions": {
    "include": [
      "wwwroot",
      "web.config"
    ]
  },

  "scripts": {
    "postpublish": [ "dotnet publish-iis --publish-folder %publish:OutputPath% --framework %publish:FullTargetFramework%" ]
  }
}</pre>

What we are adding is MVC, because we will need it for setting up the API, some logging things and AutoMappers to map between our DTOs and our real Entities. We wont use it in this tutorial, but for a kickstart its good to have them.

Visual Studio should now go ahead and resolve, download and install these packages.

&nbsp;

Now go on and add an "app"-Folder, a "js"-folder and a "css"-folder to the wwwroot-folder.

<blockquote>Everything in the wwwroot-folder is going to be served to the client. This is where our app lives. You can also build your application outside this folder and copy the relevant files in it via gulp etc. I am not doing this in this case because its easier to debug in the when launching. If you have a better approach I would love to hear it :)</blockquote>

<a href="http://offering.solutions/wp-content/uploads/2016/01/04.png" rel="attachment wp-att-945"><img class="aligncenter size-full wp-image-945" src="http://offering.solutions/wp-content/uploads/2016/01/04.png" alt="04" width="369" height="260" /></a>

To get started with our web application we need an index.html. Lets add it and put nothing in it so far.

<a href="http://offering.solutions/wp-content/uploads/2016/01/05.png" rel="attachment wp-att-946"><img class="aligncenter size-full wp-image-946" src="http://offering.solutions/wp-content/uploads/2016/01/05.png" alt="05" width="1928" height="1168" /></a>

<h2>The Serverside/API</h2>

Lets get started with the serverside: To wire up everything we need to configure our application in the Startup.cs.

<pre class="lang:c# decode:true">using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Logging;

namespace ASPNETCoreAngular2Demo
{
    public class Startup
    {
        public IConfigurationRoot Configuration { get; }

        // This method gets called by the runtime. Use this method to add services to the container.
        // For more information on how to configure your application, visit http://go.microsoft.com/fwlink/?LinkID=398940
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddCors();
            services.AddMvc();
        }

        public Startup(IHostingEnvironment env)
        {
            var builder = new ConfigurationBuilder()
                .SetBasePath(env.ContentRootPath)
                .AddJsonFile("appsettings.json", optional: true, reloadOnChange: true)
                .AddJsonFile($"appsettings.{env.EnvironmentName}.json", optional: true);

            builder.AddEnvironmentVariables();
            Configuration = builder.Build();
        }


        // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
        public void Configure(IApplicationBuilder app, ILoggerFactory loggerFactory)
        {
            loggerFactory.AddConsole(Configuration.GetSection("Logging"));
            loggerFactory.AddDebug();

            app.UseCors(config =&gt;
                config.AllowAnyHeader()
                    .AllowAnyMethod()
                    .AllowAnyOrigin());

            app.UseDefaultFiles();
            app.UseStaticFiles();

            app.UseMvc();
        }
    }
}
</pre>

Because at the beginning of this page we are consuming an "appsettings.json" we should also create it. Create a new file called like this beside the Startup.cs and paste some logging information to it. You can add some informations you want to have in there later if you want. For the start, this should be okay.

<pre class="lang:js decode:true">{
    "Logging": {
        "IncludeScopes": false,
        "LogLevel": {
            "Default": "Debug",
            "System": "Information",
            "Microsoft": "Information"
        }
    }
}</pre>

Our application should now be able to start up. Just go to the commandline and type "dotnet run" in it. This will start a webserver and serve all files in wwwroot.

<a href="http://offering.solutions/wp-content/uploads/2016/01/AspNetCoreAngular2Demo_3.jpg"><img class="aligncenter size-full wp-image-1597" src="http://offering.solutions/wp-content/uploads/2016/01/AspNetCoreAngular2Demo_3.jpg" alt="AspNetCoreAngular2Demo_3" width="749" height="167" /></a>

<a href="http://offering.solutions/wp-content/uploads/2016/01/07.png" rel="attachment wp-att-950"><img class="aligncenter size-full wp-image-950" src="http://offering.solutions/wp-content/uploads/2016/01/07.png" alt="07" width="690" height="291" /></a>

Great so far! Now we can add a controller!

For this go ahead in the root of the project and add a folder "Controller" which wil hold all the controllers for us. Then you can add an standardcontroller for the first run.

<blockquote>I like to stay to IActionResult-Interface in my controllers. It just gives me a way I am used to and is clearer for me. You do not have to do this. The controller is just for demo purposes.</blockquote>

<a href="http://offering.solutions/wp-content/uploads/2016/01/08.png" rel="attachment wp-att-952"><img class="aligncenter size-full wp-image-952" src="http://offering.solutions/wp-content/uploads/2016/01/08.png" alt="08" width="1928" height="1168" /></a>If we now run the server again with "dotnet run" it will start and via Postman we can check if everything works correctly:

<a href="http://offering.solutions/wp-content/uploads/2016/01/09.png" rel="attachment wp-att-953"><img class="aligncenter wp-image-953 size-full" src="http://offering.solutions/wp-content/uploads/2016/01/09.png" alt="Angular 2 and ASP.NET Core 1.0 Starter" width="1415" height="838" /></a>

&nbsp;

<h2>The Client</h2>

&nbsp;

Great. So lets start clientside. What we need is an AngularJs 2.0 environment. For the tooling I made the best experiences using <a href="https://code.visualstudio.com/">Visual Studio Code</a>.

We already defined the wwwroot-Folder which holds every file which is transported to the client.

<a href="http://offering.solutions/wp-content/uploads/2016/01/11.png" rel="attachment wp-att-1017"><img class="aligncenter size-full wp-image-1017" src="http://offering.solutions/wp-content/uploads/2016/01/11.png" alt="11" width="254" height="120" /></a>

As we start developing AngularJs 2.0 we have to prepare our environment for working with Typescript. I already have an instruction how to start in a previous Blogpost <a href="http://offering.solutions/articles/asp-net/getting-started-with-visual-studio-code-and-typescript/">Getting started with Visual Studio Code &amp; Typescript</a>. But lets cover some thing in short to get it customised for this AngularJs 2.0 case.

Lets add a file for configuring the typescript-options for our project: tsconfig.json to the root of our project.

<pre class="lang:js decode:true">{
 "compilerOptions": {
    "target": "es5",
    "module": "commonjs",
    "moduleResolution": "node",
    "sourceMap": true,
    "emitDecoratorMetadata": true,
    "experimentalDecorators": true,
    "removeComments": false,
    "noImplicitAny": false
  },
  "compileOnSave": true,
  "exclude": [
    "node_modules",
    "wwwroot/js",
    "typings/main",
    "typings/main.d.ts"
  ]
}</pre>

<h3>Adding Angular2</h3>

AngularJs 2 is served via npm. To get Angular2 we have to add a package.json to the root of our project. "npm start" will run the lite-server, compile the *.ts files and start the watcher :)

<pre class="lang:c# decode:true">{
  "name": "ng2app",
  "version": "0.0.0",
  "license": "MIT",
  "angular-cli": {},
  "scripts": {
    "start": "tsc &amp;&amp; gulp get:started &amp;&amp; concurrently \"npm run tsc:w\" \"dotnet run\" \"npm run lite\" ",
    "lite": "lite-server",
    "postinstall": "typings install",
    "tsc": "tsc",
    "tsc:w": "tsc -w",
    "typings": "typings",
    "cleanup": "rimraf node_modules &amp;&amp; rimraf typings"
  },
  "private": true,
  "dependencies": {
    "@angular/common": "2.0.0",
    "@angular/compiler": "2.0.0",
    "@angular/core": "2.0.0",
    "@angular/forms": "2.0.0",
    "@angular/http": "2.0.0",
    "@angular/platform-browser": "2.0.0",
    "@angular/platform-browser-dynamic": "2.0.0",
    "@angular/router": "3.0.0",
    "@angular/upgrade": "2.0.0",
    "angular2-toaster": "^1.0.1",
    "bootstrap": "^3.3.6",
    "core-js": "^2.4.1",
    "jquery": "2.2.4",
    "reflect-metadata": "^0.1.3",
    "rxjs": "5.0.0-beta.12",
    "systemjs": "0.19.27",
    "zone.js": "^0.6.23"
  },
  "devDependencies": {
    "del": "^2.2.0",
    "gulp": "^3.9.1",
    "rimraf": "^2.5.2",
    "run-sequence": "^1.1.5",
    "concurrently": "^2.2.0",
    "lite-server": "^2.2.2",
    "typescript": "^2.0.2",
    "typings": "^1.3.2"
  }
}</pre>

or just type "npm init" to answer all questions and get an project.json file generated.

After this you can install AngularJs 2 and all other dependencies via "npm install" or Visual Studio will do this for you automatically. Just just have to wait a few seconds. It will add a node_modules folder to your root application and copy all files, also the *.d.ts-files you need to develop with typescript.

<h4>System.js</h4>

Now we have to configure the system.js as so:

<pre class="lang:js decode:true ">(function (global) {
  System.config({
    paths: {
      // paths serve as alias
      'npm:': 'js/'
    },
    // map tells the System loader where to look for things
    map: {
      // our app is within the app folder
      app: 'app',
      // angular bundles
      '@angular/core': 'npm:@angular/core/bundles/core.umd.js',
      '@angular/common': 'npm:@angular/common/bundles/common.umd.js',
      '@angular/compiler': 'npm:@angular/compiler/bundles/compiler.umd.js',
      '@angular/platform-browser': 'npm:@angular/platform-browser/bundles/platform-browser.umd.js',
      '@angular/platform-browser-dynamic': 'npm:@angular/platform-browser-dynamic/bundles/platform-browser-dynamic.umd.js',
      '@angular/http': 'npm:@angular/http/bundles/http.umd.js',
      '@angular/router': 'npm:@angular/router/bundles/router.umd.js',
      '@angular/forms': 'npm:@angular/forms/bundles/forms.umd.js',
      'angular2-toaster': 'npm:angular2-toaster',
      // other libraries
      'rxjs': 'npm:rxjs'
    },
    // packages tells the System loader how to load when no filename and/or no extension
    packages: {
      app: {
        main: './main.js',
        defaultExtension: 'js'
      },
      rxjs: {
        defaultExtension: 'js'
      },
      'angular2-toaster': {
        defaultExtension: 'js'
      }
    }
  });
})(this);</pre>

Now go ahead in you index.html and link to the files in the client js folder:

<pre class="lang:xhtml decode:true">&lt;!DOCTYPE html&gt;
&lt;html&gt;

&lt;head&gt;
    &lt;base href="." /&gt;
    &lt;title&gt;ASP.NET Core 1.0 RC 2 Angular 2 Demo&lt;/title&gt;

    &lt;!-- inject:css --&gt;
    &lt;link rel="stylesheet" href="css/bootstrap.css"&gt;
    &lt;!-- endinject --&gt;
&lt;/head&gt;

&lt;body&gt;
    &lt;my-app&gt;Loading...&lt;/my-app&gt;

     &lt;!-- 1. Load libraries --&gt;
    &lt;script src="js/shim.min.js"&gt;&lt;/script&gt;
    &lt;script src="js/zone.js"&gt;&lt;/script&gt;
    &lt;script src="js/Reflect.js"&gt;&lt;/script&gt;
    &lt;script src="js/system.src.js"&gt;&lt;/script&gt;
    
    &lt;script src="js/jquery.js"&gt;&lt;/script&gt;
    
    &lt;script src="js/bootstrap.js"&gt;&lt;/script&gt;
    
    &lt;!-- 2. Configure SystemJS --&gt;
    &lt;script src="system.config.js"&gt;&lt;/script&gt;
    &lt;script&gt;
      System.import('app').catch(function(err){ console.error(err); });
    &lt;/script&gt;

&lt;/body&gt;

&lt;/html&gt;</pre>

&nbsp;

<h2>Finally...</h2>

We are done so far:

You can now go ahead and develop you application in Typescript and Angular2. You could start with the <a href="https://angular.io/docs/ts/latest/quickstart.html">AngularJs 2</a> quickstart. All files you need should be there. Its important to add the <span class="css-truncate css-truncate-target"><a id="6427dc610ec40c788cba04a9c39d6dcf-3a6c754786e23660551a464c4f7424b4c202dc15" class="js-navigation-open" title="main.ts" href="https://github.com/FabianGosebrink/ASPNET-Core-Angular2-StarterTemplate/blob/master/src/ASPNETCoreAngular2Demo/wwwroot/app/main.ts">main.ts</a></span>, the <span class="css-truncate css-truncate-target"><a id="797cf0e0433d30aafe126d8078bbba22-2a0ac2a711d89bed6c6ee87c1c0aad66d94c13f6" class="js-navigation-open" title="app.component.ts" href="https://github.com/FabianGosebrink/ASPNET-Core-Angular2-StarterTemplate/blob/master/src/ASPNETCoreAngular2Demo/wwwroot/app/app.component.ts">app.component.ts</a></span> and <span class="css-truncate css-truncate-target"><a id="7bf9078f6a063b5fd77a92650e14ed10-9e572676ffcccc9c73d851c7c22a034eea7b4c83" class="js-navigation-open" title="app.module.ts" href="https://github.com/FabianGosebrink/ASPNET-Core-Angular2-StarterTemplate/blob/master/src/ASPNETCoreAngular2Demo/wwwroot/app/app.module.ts">app.module.ts</a></span> to get an entry point and bootstrapping your application.

<blockquote>If you, like me, always forget to compile the typescript files: open a cmd in the app folder and just type "npm run tsc:w". This activates a typescript watcher using the tsconfig in the root of the project and will always compile the *.ts files into *.js files if there are some changes</blockquote>

<h3>Further steps:</h3>

You could now start using gulp/grunt to copy files to the locations automatically instead of doing it manually. Or you could start consuming the api using the http-Service like shown <a href="https://auth0.com/blog/2015/10/15/angular-2-series-part-3-using-http/">here</a>. I have done that all in my example repository here on Github. Hope you like it.

<a href="https://github.com/FabianGosebrink/ASPNET-Core-Angular2-StarterTemplate">https://github.com/FabianGosebrink/ASPNET-Core-Angular2-StarterTemplate</a>

&nbsp;

Regards &amp; HTH

Fabian

&nbsp;
