---
id: 1626
title: How to set up Angular 2 and Webpack in Visual Studio with ASP.NET Core
date: 2016-08-27T16:58:32+00:00
author: Fabian Gosebrink
layout: post
tags: aspnet aspnetcore Visual Studio 2015 webpack dev server
logo: 'assets/images/logo_small.png'
navigation: True
cover: 'assets/images/download_edit_dark.jpg'
subclass: 'post tag-speeches'
disqus: true
categories: articles
---

With this blogpost I want to show an example of how to set up Angular 2 and Webpack in Visual Studio with ASP.NET Core ready for dev and production builds.

If you read this until the end you will have a picture how to enable webpack and webpack-dev-server with npm and how to use webpack to bundle your angular 2 application and inject files into index.html for your web application.

I had this topic already with my colleague [DamienBod](https://github.com/damienbod)
  
[With this blogpost I want to show an example of how to set up Angular 2 and Webpack in Visual Studio with ASP.NET Core ready for dev and production builds.

If you read this until the end you will have a picture how to enable webpack and webpack-dev-server with npm and how to use webpack to bundle your angular 2 application and inject files into index.html for your web application.

I had this topic already with my colleague [DamienBod](https://github.com/damienbod)
  
](https://github.com/damienbod/Angular2WebpackVisualStudio) and a StarterTemplate [here](https://github.com/FabianGosebrink/ASPNET-Core-Angular2-StarterTemplate) (which is using system.js as bootstrapper for your application), but this post is more to show how we get there and which steps you should take to get things going. For Damien and me this was a bit confusing in the beginning so this is the guide how we started actually. I hope you like reading it as much as I liked writing it.

Find the code here:

<https://github.com/FabianGosebrink/ASPNET-Core-Angular2-Webpack-StarterTemplate>

### The new project

Select “File" -> "New Project”

![How to set up Angular 2 and Webpack in Visual Studio with ASP.NET Core]({{site.baseurl}}assets/articles/2016-08-27/Clipboard01.jpg)

And create a new project. In this case we name it “Angular2WebpackStarter”. After creating you can take the empty template like this:

![How to set up Angular 2 and Webpack in Visual Studio with ASP.NET Core]({{site.baseurl}}assets/articles/2016-08-27/Clipboard02.jpg)

Make sure you have installed the latest typescript nuget package because we will need it later to work with angular2 and visual studio 2015. A Resharper in the latest version also cannot be a mistake J

### Add the necessary files and folders

The next step is to add the files and folders which are needed to keep your application structured and organized.

Add a new folder in your solution and call it “Angular2app”. This where all the client related dependencies are stored and being “compiled” and transferred to the wwwroot folder later on.

Next, add these files to your project and fill it with this data:

&nbsp;

**package.json**
  
Your npm file related to your project. It keeps als dependencies and everything for your client.

<pre class="lang:js decode:true ">{
  "name": "aspnetcoreangular2webpackstarter",
  "version": "0.0.0",
  "license": "MIT",
  "scripts": {
    "start": "tsc && npm install && npm run build && \"dotnet run\" ",
    "startWebpackDevServer": "webpack-dev-server --inline --progress --port 8080",
    "build": "SET NODE_ENV=development && webpack -d --color && dotnet run",
    "buildProduction": "SET NODE_ENV=production && webpack -d --color",
    "lint": "tslint ./angular2app/**/*.ts -t verbose",
    "postinstall": "typings install",
    "tsc": "tsc",
    "tsc:w": "tsc -w",
    "typings": "typings"
  },
  "keywords": [],
  "author": "Fabian Gosebrink",
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
    "tslint": "^3.7.4",
    "lodash": "^4.11.1",
    "rimraf": "^2.5.2",
    "node-sass": "3.8.0",
    "concurrently": "^2.2.0",
    "lite-server": "^2.2.2",
    "typescript": "^2.0.2",
    "typings": "^1.3.2"
  }
}</pre>

&nbsp;

**tsconfig.json**

Is configuring your tsc compiler. Whenever your run the “tsc” command from the commandline it will be taken as configuration for the typescript-compiler.

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

**typings.json**

Gets your global typing dependencies needed for angular2

<pre class="lang:js decode:true">{
  "globalDependencies": {
    "core-js": "registry:dt/core-js#0.0.0+20160725163759",
    "jasmine": "registry:dt/jasmine#2.2.0+20160621224255",
    "node": "registry:dt/node#6.0.0+20160909174046"
  }
}</pre>

So your solution should look something like this:

![How to set up Angular 2 and Webpack in Visual Studio with ASP.NET Core]({{site.baseurl}}assets/articles/2016-08-27/Zwischenablage01.jpg)

### Fill the angular2app-folder

Next we are going to fill the angular2 app folder. Here we are going to be lazy. We are taking the [quickstart repo from angular.io](https://github.com/angular/quickstart) and copy the files we need form there. For the sake of simplicity we will only take the [app.module](https://github.com/angular/quickstart/tree/master/app) to get things going. To be structured its best you create an app folder inside to place your code there.

![How to set up Angular 2 and Webpack in Visual Studio with ASP.NET Core]({{site.baseurl}}assets/articles/2016-08-27/Zwischenablage02.jpg)

But to start we need an index.html. And for webpack to be prepared we need an entrypoint for the polyfills and the vendor files we need for our app. Lets add these files.

index.html

<pre class="lang:xhtml decode:true ">&lt;html&gt;
&lt;head&gt;
    &lt;title&gt;Angular2WebpackStarter&lt;/title&gt;
    &lt;meta charset=UTF-8&gt;
    &lt;meta name=viewport content="width=device-width,initial-scale=1"&gt;
&lt;/head&gt;
&lt;body&gt;
    &lt;my-app&gt;Loading...&lt;/my-app&gt;
&lt;/body&gt;
&lt;/html&gt;</pre>

> Note that we include **nothing** here yet. This is on purpose. Read further&#8230; 🙂

I reduced the vendor.ts and polyfills.ts to store only the things we need

polyfills.ts

<pre class="lang:js decode:true ">import "core-js/es6";
import "core-js/es7/reflect";
require("zone.js/dist/zone");</pre>

vendor.ts

<pre class="lang:js decode:true">// RxJS.
import "rxjs";

// Angular 2.
import "@angular/common";
import "@angular/compiler";
import "@angular/core";
import "@angular/http";
import "@angular/platform-browser";
import "@angular/platform-browser-dynamic";
import "@angular/router";

// Reflect Metadata.
import "reflect-metadata";

// Other vendors for example jQuery, Lodash or Bootstrap
// You can import js, ts, css, sass, ...

import "jquery";
import "bootstrap/dist/js/bootstrap";
import "bootstrap/dist/css/bootstrap.min.css";
import "angular2-toaster/lib/toaster.css";
import "angular2-toaster/angular2-toaster";</pre>

&nbsp;

So you solution should now look a little something like this:

![How to set up Angular 2 and Webpack in Visual Studio with ASP.NET Core]({{site.baseurl}}assets/articles/2016-08-27/Zwischenablage03.jpg)

Why did we all this and if the wwwroot-folder is the one to be served to the client&#8230;why are we doing all this?

This is where webpack comes into play. We will configure webpack now to build our application into the wwwroot folder. Let&#8217;s do this&#8230;

<p class="projectTitle">
  Lets add a webpack.config.js file to the root of the project. (Make sure you have the <a href="https://visualstudiogallery.msdn.microsoft.com/5497fd10-b1ba-474c-8991-1438ae47012a">WebPack Task Runner Extension</a> installed) .
</p>

<p class="projectTitle">
  So&#8230;you added the file: Paste the following content:
</p>

<pre class="lang:js decode:true ">var ExtractTextPlugin = require("extract-text-webpack-plugin");
var webpack = require("webpack");
var HtmlWebpackPlugin = require("html-webpack-plugin");
var CleanWebpackPlugin = require('clean-webpack-plugin');

module.exports = {
    entry: {
        "polyfills": "./angular2App/polyfills.ts",
        "vendor": "./angular2App/vendor.ts",
        "app": "./angular2App/app/main.ts"
    },
    resolve: {
        extensions: ['', '.ts', '.js', '.json', '.css', '.scss', '.html']
    },
    output: {
        path: "./wwwroot",
        filename: "js/[name]-[hash:8].bundle.js"
    },

    module: {
        loaders: [
            {
                test: /\.ts$/,
                loader: "ts"
            },
            {
                test: /\.html$/,
                loader: "html"
            },
            {
                test: /\.(png|jpg|gif|ico|woff|woff2|ttf|svg|eot)$/,
                loader: "file?name=assets/[name]-[hash:6].[ext]",
            },

            // Load css files which are required in vendor.ts
            {
                test: /\.css$/,
                loader: ExtractTextPlugin.extract('style', 'css')
            }
        ]
    },
    plugins: [
        new ExtractTextPlugin("css/[name]-[hash:8].bundle.css"),
        new webpack.optimize.CommonsChunkPlugin({
            name: ["app", "vendor", "polyfills"]
        }),
       
        new webpack.ProvidePlugin({
            jQuery: 'jquery',
            $: 'jquery',
            jquery: 'jquery'
        })
    ],
    devServer: {
        historyApiFallback: true,
        stats: "minimal"
    }
};</pre>

&nbsp;

&nbsp;

This file is no magic, don&#8217;t be scared:

First we require everything we need to use to kick off webpack. Then we will export our complete configuration. Our entrypoints are pointing to the files we just created (remember?) and of course our entrypoint of our application.

The "resolve"-array tells webpack to look for those file endings. The "output" is what we all were looking for. Here we tell webpack "Hey, what no matter what you are doing and how you are doing it, put it in the ./wwwroot-Folder and please be gentle and name the files like I will tell you later on ([name]) and please put a hash on it at the end, but only 8 digits ([hash:8])". This is it.

Loaders are telling webpack **how** to handle such file endings. So: If you encounter such a file, so this and that. The plugins are telling webpack how to behave generally, to point which files out etc. And this is alle the magic.

&nbsp;

**package.json**

Add the webpack-things we need to the "DevDependencies"-section in the package.json:

<pre class="lang:js decode:true ">"ts-loader": "^0.8.1",
        "extract-text-webpack-plugin": "^1.0.1",
        "file-loader": "^0.8.5",
        "webpack": "^1.12.14",
        "webpack-dev-server": "^1.14.1",
        "webpack-merge": "^0.9.0",
        "webpack-stream": "^3.2.0",
        "html-loader": "^0.4.3",
        "html-webpack-plugin": "^2.15.0"</pre>

Run npm install or let VS do this for you.

&nbsp;

&nbsp;

Now that you&#8217;ve done this open up the Task Runner Explorer in Visual Studio and let the "Run &#8211; Development" go:

![How to set up Angular 2 and Webpack in Visual Studio with ASP.NET Core]({{site.baseurl}}assets/articles/2016-08-27/Zwischenablage04.jpg)

Et voila: our wwwroot-folder got filled with our entire app (or with what we have got so far). But there is one thing missing: The index.html!

Lets tell webpack to take our index.html we already have got and inject the sources into it and copy it too. To do this extend the "plugins"-section like this:

<pre class="lang:js decode:true ">plugins: [
        new ExtractTextPlugin("[name].bundle.css"),
        new webpack.optimize.CommonsChunkPlugin({
            name: ["app", "vendor", "polyfills"]
        }),

        // inject in index.html
        new HtmlWebpackPlugin({
            template: "./angular2App/index.html",
            inject: "body"
        })
    ],</pre>

and add the

<pre class="lang:js decode:true ">var HtmlWebpackPlugin = require("html-webpack-plugin");</pre>

at the top of the page. We need to include what we want to use 😉

let it run again:

![How to set up Angular 2 and Webpack in Visual Studio with ASP.NET Core]({{site.baseurl}}assets/articles/2016-08-27/Zwischenablage05.jpg)

Lets take a look into this index.html:

<pre class="lang:xhtml decode:true ">&lt;html&gt;
&lt;head&gt;
    &lt;title&gt;Angular2WebpackStarter&lt;/title&gt;
    &lt;meta charset=UTF-8&gt;
    &lt;meta name=viewport content="width=device-width,initial-scale=1"&gt;
&lt;/head&gt;
&lt;body&gt;
    &lt;my-app&gt;Loading...&lt;/my-app&gt;
    &lt;script type="text/javascript" src="polyfills-54447aaf.bundle.js"&gt;&lt;/script&gt;
    &lt;script type="text/javascript" src="vendor-54447aaf.bundle.js"&gt;&lt;/script&gt;
    &lt;script type="text/javascript" src="app-54447aaf.bundle.js"&gt;&lt;/script&gt;
&lt;/body&gt;
&lt;/html&gt;</pre>

our files got injected!

Before we actually see our application we have to modify the Startup.cs to behave like we want it to:

Add

<pre class="lang:js decode:true ">"Microsoft.AspNetCore.Mvc": "1.0.0",
    "Microsoft.AspNetCore.StaticFiles": "1.0.0"</pre>

to your project.json and in the Startup write:

<pre class="lang:c# decode:true">public class Startup
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
    }</pre>

Now we have an index.html and we can press the play-button in Visual Studio:

![How to set up Angular 2 and Webpack in Visual Studio with ASP.NET Core]({{site.baseurl}}assets/articles/2016-08-27/Zwischenablage06.jpg)

So it is working!!

Great so far.

### Adding webpack dev server:

Wouldn&#8217;t it be nice to get the files refreshed as soon as I start working on them with this webpack construction? Well we already added the webpack dev-server in the package.json as dependency and in the web.config we already configured it in a basic way. So now lets use it:

You can now change the "npm start" command but we will simply make another one like "startWebpackDevServer". So add the following line into the scripts section of package.json:

<pre class="lang:js decode:true">"startWebpackDevServer": "webpack-dev-server --inline --progress --port 8080",</pre>

Now open a commandline to the level of your package.json and type "npm run startWebpackDevServer":

![How to set up Angular 2 and Webpack in Visual Studio with ASP.NET Core]({{site.baseurl}}assets/articles/2016-08-27/Zwischenablage07.jpg)

Let it open and browse to "localhost:8080"

![How to set up Angular 2 and Webpack in Visual Studio with ASP.NET Core]({{site.baseurl}}assets/articles/2016-08-27/Zwischenablage08.jpg)

There your app appears!

So lets take a look how this behaves if we change a file

![How to set up Angular 2 and Webpack in Visual Studio with ASP.NET Core]({{site.baseurl}}assets/articles/2016-08-27/Animation.gif)

Okay great.

Now lets go ahead and build this thing up to use development and production builds, which is our last step:

### Development and Production builds

Lets take a closer look to what we did when we first ran our webpack:

![How to set up Angular 2 and Webpack in Visual Studio with ASP.NET Core]({{site.baseurl}}assets/articles/2016-08-27/Zwischenablage09.jpg)

There webpack is setting the variable "NODE_ENV" to "development". Lets use that!!!

rename the "webpack.config.js" to "webpack.dev.js". Create a new webpack.config.js.

All we need to do now is creating a switch which says "if you are development, use a (new) file (we will create) which takes the dev-thing, otherwise take the production ones"

<pre class="lang:js decode:true ">var environment = (process.env.NODE_ENV || "development").trim();

if (environment === "development") {
    module.exports = require('./webpack.dev.js');
} else {
    module.exports = require('./webpack.prod.js');
}</pre>

So THIS is our new entrypoint for webpack, the new webpack.config.json!!

Create a file which is called "webpack.prod.js" which will be our production file in the future. You should now have something like this:

![How to set up Angular 2 and Webpack in Visual Studio with ASP.NET Core]({{site.baseurl}}assets/articles/2016-08-27/Zwischenablage10.jpg)

To test this put a simple console.log in the production file like:

<pre class="lang:js decode:true ">console.log("----&gt; Production");</pre>

and let it run:

![How to set up Angular 2 and Webpack in Visual Studio with ASP.NET Core]({{site.baseurl}}assets/articles/2016-08-27/Zwischenablage11.jpg)

So now lets pimp the production a bit. Its mostly the same we have so far for dev, but we need a little more plugins. Maybe the one to uglify the js [UglifyJsPlugin](http://webpack.github.io/docs/list-of-plugins.html#uglifyjsplugin) and to delete the files in the wwwroot first "[Clean for webpack](https://github.com/johnagan/clean-webpack-plugin)".

Copy the whole content from the webpack.dev.json to the prod.json and simply add the uglifyjs-plugin like this:

<pre class="lang:js decode:true ">plugins: [
       new ExtractTextPlugin("[name].bundle.css"),
       new webpack.optimize.CommonsChunkPlugin({
           name: ["app", "vendor", "polyfills"]
       }),
       
       new webpack.optimize.UglifyJsPlugin({
           compress: {
               warnings: false
           }
       }),

       // inject in index.html
       new HtmlWebpackPlugin({
           template: "./angular2App/index.html",
           inject: "body"
       })
    ],</pre>

If you now let the prod-task run the files will be treated as ever but they are uglified as well.

To clean the wwwroot-folder first we need to install another plugin "Clean for webpack" and use it:

<pre class="lang:js decode:true ">npm install clean-webpack-plugin --save-dev</pre>

and in both webpack-files

<pre class="lang:js decode:true">var CleanWebpackPlugin = require('clean-webpack-plugin');
 //....

plugins: [
       new ExtractTextPlugin("[name].bundle.css"),
       new webpack.optimize.CommonsChunkPlugin({
           name: ["app", "vendor", "polyfills"]
       }),

       new webpack.optimize.UglifyJsPlugin({
           compress: {
               warnings: false
           }
       }),
         new CleanWebpackPlugin(
            [
                "./wwwroot/"
            ]
        ),
       // inject in index.html
       new HtmlWebpackPlugin({
           template: "./angular2App/index.html",
           inject: "body"
       })
    ],</pre>

Now our folder gets cleaned. You can add this into dev, too.

This is it. This is how you can treat Angular2 with webpack in Visual Studio with an ASPNET Core application. I hope you liked and enjoyed reading.

HTH
  
Fabian