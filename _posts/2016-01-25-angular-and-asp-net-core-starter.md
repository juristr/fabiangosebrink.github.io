---
title: Angular and ASP.NET Core Starter
date: 2016-01-25 19:47
author: Fabian Gosebrink
layout: post
tags: Angular ASP.NET Core Visual Studio Code
logo: 'assets/images/logo_small.png'
navigation: True
cover: 'assets/images/aerial-view-of-laptop-notebook-mobile-phone-and-coffee-cup-on-wooden-table.jpg'
subclass: 'post tag-speeches'
disqus: true
categories: articles
---

With this bIog I want to give you an Angular and ASP.NET Core Starter.

Code here:

[https://github.com/FabianGosebrink/ASPNET-Core-Angular2-StarterTemplate](https://github.com/FabianGosebrink/ASPNET-Core-Angular2-StarterTemplate)

**Updated to the newest Version!**

Nowadays you hear a lot of new technologies around the web stuff and with this post I want to show you a get-starting-guide how to start with the following technologies in general. I want to show how to set up an environment where you can start programming ASP.NET and Angular. As I started I spent a lot of time setting up all my tools and stuff. If you want to save this time: Go ahead reading.

So I want to have an environment where I can build an Angular web application with typescript, having an api where I can call to build with ASP.NET Core. We will also use [Visual Studio Code](https://code.visualstudio.com/) to get the client-side programming going.

## Lets start:

First thing to do is setting up our Visual Studio. Create a new application and choose the ASP.NET 5 Templates. I guess they will be renamed later ;-)

![AspNetCoreAngular2Demo_1]({{site.baseurl}}assets/articles/wp-content/uploads/2016/01/AspNetCoreAngular2Demo_1.jpg)
![AspNetCoreAngular2Demo_2]({{site.baseurl}}assets/articles/wp-content/uploads/2016/01/AspNetCoreAngular2Demo_2.jpg)

You will get an empty solution with no dependencies so far. Lets go and add some: open up the "project.json" and add this:

{% highlight js %}
{
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
        "netcoreapp1.0": {}
    },

    "buildOptions": {
        "emitEntryPoint": true,
        "preserveCompilationContext": true,
        "debugType": "portable",
        "compile": {
            "include": [
                "**/*.cs"
            ],
            "exclude": [
                "**/node_modules/"
            ]
        }
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
}
{% endhighlight %}

What we are adding is MVC, because we will need it for setting up the API, some logging things and AutoMappers to map between our DTOs and our real Entities. We wont use it in this tutorial, but for a kickstart its good to have them.

Visual Studio should now go ahead and resolve, download and install these packages.

Now go on and add an "app"-Folder, a "js"-folder and a "css"-folder to the wwwroot-folder.

>Everything in the wwwroot-folder is going to be served to the client. This is where our app lives. You can also build your application outside this folder and copy the relevant files in it via gulp etc. I am not doing this in this case because its easier to debug in the when launching. If you have a better approach I would love to hear it :)
    
![04]({{site.baseurl}}assets/articles/wp-content/uploads/2016/01/04.png)

To get started with our web application we need an index.html. Lets add it and put nothing in it so far.

![05]({{site.baseurl}}assets/articles/wp-content/uploads/2016/01/05.png)


## The Serverside/API

Lets get started with the serverside: To wire up everything we need to configure our application in the Startup.cs.

{% highlight cs %}
using Microsoft.AspNetCore.Builder;
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

            app.UseCors(config =>
                config.AllowAnyHeader()
                    .AllowAnyMethod()
                    .AllowAnyOrigin());

            app.UseDefaultFiles();
            app.UseStaticFiles();

            app.UseMvc();
        }
    }
}
{% endhighlight %}

Because at the beginning of this page we are consuming an "appsettings.json" we should also create it. Create a new file called like this beside the Startup.cs and paste some logging information to it. You can add some informations you want to have in there later if you want. For the start, this should be okay.

{% highlight js %}
{
    "Logging": {
        "IncludeScopes": false,
        "LogLevel": {
            "Default": "Debug",
            "System": "Information",
            "Microsoft": "Information"
        }
    }
}
{% endhighlight %}

Our application should now be able to start up. Just go to the commandline and type "dotnet run" in it. This will start a webserver and serve all files in wwwroot.

![AspNetCoreAngular2Demo_3]({{site.baseurl}}assets/articles/wp-content/uploads/2016/01/AspNetCoreAngular2Demo_3.jpg)

![07]({{site.baseurl}}assets/articles/wp-content/uploads/2016/01/07.png)

Great so far! Now we can add a controller!

For this go ahead in the root of the project and add a folder "Controller" which wil hold all the controllers for us. Then you can add an standardcontroller for the first run.

>I like to stay to IActionResult-Interface in my controllers. It just gives me a way I am used to and is clearer for me. You do not have to do this. The controller is just for demo purposes.

![08]({{site.baseurl}}assets/articles/wp-content/uploads/2016/01/08.png)

If we now run the server again with "dotnet run" it will start and via Postman we can check if everything works correctly:

![Angular and ASP.NET Core Starter]({{site.baseurl}}assets/articles/wp-content/uploads/2016/01/09.png)


## The Client
    
Great. So lets start clientside. What we need is an Angular environment. For the tooling I made the best experiences using [Visual Studio Code](https://code.visualstudio.com/).

We already defined the wwwroot-Folder which holds every file which is transported to the client.

![11]({{site.baseurl}}assets/articles/wp-content/uploads/2016/01/11.png)

As we start developing Angular we have to prepare our environment for working with Typescript. I already have an instruction how to start in a previous Blogpost [Getting started with Visual Studio Code & Typescript](http://offering.solutions/articles/asp-net/getting-started-with-visual-studio-code-and-typescript/). But lets cover some thing in short to get it customised for this Angular case.

Lets add a file for configuring the typescript-options for our project: tsconfig.json to the root of our project.

{% highlight js %}
{
  "compilerOptions": {
    "target": "es5",
    "module": "commonjs",
    "moduleResolution": "node",
    "sourceMap": true,
    "emitDecoratorMetadata": true,
    "experimentalDecorators": true,
    "lib": [
      "es2015",
      "dom"
    ],
    "noImplicitAny": true,
    "suppressImplicitAnyIndexErrors": true
  },
  "exclude": [
    "node_modules",
    "wwwroot/**/*"
  ]
}
{% endhighlight %}

### Adding Angular

Angular is served via npm. To get Angular we have to add a package.json to the root of our project. "npm start" will run the lite-server, compile the *.ts files and start the watcher :)

{% highlight js %}
{
  "name": "ng2app",
  "version": "0.0.0",
  "license": "MIT",
  "angular-cli": {},
  "scripts": {
    "start": "tsc && gulp get:started && concurrently \"npm run tsc:w\" \"dotnet run\" \"npm run lite\" \"gulp start-watch\"",
    "lite": "lite-server",
    "tsc": "tsc",
    "tsc:w": "tsc -w",
    "lint": "tslint ./angular2app/**/*.ts -t verbose"
  },
  "private": true,
  "dependencies": {
    "@angular/common": "~2.4.0",
    "@angular/compiler": "~2.4.0",
    "@angular/core": "~2.4.0",
    "@angular/forms": "~2.4.0",
    "@angular/http": "~2.4.0",
    "@angular/platform-browser": "~2.4.0",
    "@angular/platform-browser-dynamic": "~2.4.0",
    "@angular/router": "~3.4.0",
    "angular-in-memory-web-api": "~0.2.2",
    "angular2-toaster": "^1.0.1",
    "bootstrap": "^3.3.7",
    "core-js": "^2.4.1",
    "jquery": "2.2.4",
    "ng2-loading-bar": "0.0.6",
    "reflect-metadata": "^0.1.8",
    "rxjs": "5.0.1",
    "systemjs": "0.19.40",
    "zone.js": "^0.7.4"
  },
  "devDependencies": {
    "@types/jasmine": "^2.5.36",
    "@types/node": "^6.0.46",
    "canonical-path": "0.0.2",
    "concurrently": "^3.1.0",
    "gulp": "^3.9.1",
    "http-server": "^0.9.0",
    "lite-server": "^2.2.2",
    "lodash": "^4.16.4",
    "protractor": "~4.0.14",
    "rimraf": "^2.5.4",
    "run-sequence": "^1.2.2",
    "tslint": "^3.15.1",
    "typescript": "~2.0.10"
  }
}
{% endhighlight %}

or just type "npm init" to answer all questions and get an project.json file generated.

After this you can install Angular and all other dependencies via "npm install" or Visual Studio will do this for you automatically. Just just have to wait a few seconds. It will add a node_modules folder to your root application and copy all files, also the *.d.ts-files you need to develop with typescript.

#### System.js
    
    

Now we have to configure the system.js as so:

{% highlight js %}
/**
 * System configuration for Angular 2 samples
 * Adjust as necessary for your application needs.
 */
(function (global) {
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

      // Third Party
      'angular2-toaster': 'npm:angular2-toaster',
      // 'ng2-slim-loading-bar': 'npm:ng2-slim-loading-bar/bundles/index.umd.js',
      "ng2-loading-bar": "npm:ng2-loading-bar",

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
      },
      'ng2-slim-loading-bar': {
        defaultExtension: 'js'
      },
      "ng2-loading-bar": { "main": "index.js", "defaultExtension": "js" }
    }
  });
})(this);
{% endhighlight %}

Now go ahead in you index.html and link to the files in the client js folder:

{% highlight html %}
<!DOCTYPE html>
<html>

<head>
    <base href="/" />
    <title>ASP.NET Core 1.0 Angular 2 Demo</title>

    <link rel="stylesheet" type="text/css" href="css/bootstrap.css">
    <link rel="stylesheet" type="text/css" href="css/toaster.css" />
    <link rel="stylesheet" type="text/css" href="css/style.css" />
</head>

<body>
    <my-app>Loading...</my-app>

     <!-- 1. Load libraries -->
    <script src="js/shim.min.js"></script>
    <script src="js/zone.js"></script>
    <script src="js/Reflect.js"></script>
    <script src="js/system.src.js"></script>
    
    <script src="js/jquery.js"></script>
    
    <script src="js/bootstrap.js"></script>
    
    <!-- 2. Configure SystemJS -->
    <script src="system.config.js"></script>
    <script>
      System.import('app').catch(function(err){ console.error(err); });
    </script>

</body>

</html>
{% endhighlight %}

## Finally...

We are done so far:

You can now go ahead and develop you application in Typescript and Angular. You could start with the [Angular](https://angular.io/docs/ts/latest/quickstart.html) quickstart. All files you need should be there. Its important to add the [main.ts](https://github.com/FabianGosebrink/ASPNET-Core-Angular2-StarterTemplate/blob/master/src/ASPNETCoreAngular2Demo/wwwroot/app/main.ts) , the [app.component.ts](https://github.com/FabianGosebrink/ASPNET-Core-Angular2-StarterTemplate/blob/master/src/ASPNETCoreAngular2Demo/wwwroot/app/app.component.ts) and [app.module.ts](https://github.com/FabianGosebrink/ASPNET-Core-Angular2-StarterTemplate/blob/master/src/ASPNETCoreAngular2Demo/wwwroot/app/app.module.ts) to get an entry point and bootstrapping your application.

>If you, like me, always forget to compile the typescript files: open a cmd in the app folder and just type "npm run tsc:w". This activates a typescript watcher using the tsconfig in the root of the project and will always compile the *.ts files into *.js files if there are some changes

### Further steps:

You could now start using gulp/grunt to copy files to the locations automatically instead of doing it manually. Or you could start consuming the api using the http-Service like shown [here](https://auth0.com/blog/2015/10/15/angular-2-series-part-3-using-http/). I have done that all in my example repository here on Github. Hope you like it.

[https://github.com/FabianGosebrink/ASPNET-Core-Angular2-StarterTemplate](https://github.com/FabianGosebrink/ASPNET-Core-Angular2-StarterTemplate)

Regards & HTH

Fabian
