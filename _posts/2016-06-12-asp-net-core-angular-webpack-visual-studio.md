---
title: ASP.NET Core and Angular with Webpack and Visual Studio
date: 2016-06-12 22:10
author: Fabian Gosebrink
layout: post
tags: Angular ASP.NET Core webpack
logo: 'assets/images/logo_small.png'
navigation: True
cover: 'assets/images/aerial-view-of-laptop-notebook-mobile-phone-and-coffee-cup-on-wooden-table.jpg'
subclass: 'post tag-speeches'
disqus: true
categories: articles
---

This article shows how to use ASP.NET Core and Angular with webpack and Visual Studio. Both the client and the server side of the application is implemented inside one ASP.NET Core project which makes it easier to deploy.

![vs_webpack_angular2](http://offering.solutions/wp-content/uploads/2016/06/vs_webpack_angular2.png)

**Code:** [https://github.com/damienbod/Angular2WebpackVisualStudio](https://github.com/damienbod/Angular2WebpackVisualStudio)

**Authors** *Fabian Gosebrink, Damien Bowden*.
This post is hosted on both [http://damienbod.com](http://damienbod.com) and [http://offering.solutions/](http://offering.solutions/) and will be hosted on [http://blog.noser.com](http://blog.noser.com) afterwards.

**Setting up the application**

The ASP.NET Core application contains both the server side API services and also hosts the Angular 2 client application. The source code for the Angular 2 application is implemented in the angular2App folder. Webpack is then used to deploy the application, using the development build or a production build, which deploys the application to the wwwroot folder. This makes it easy to deploy the application using the standard tools from Visual Studio with the standard configurations.

**npm configuration**

The npm package.json configuration loads all the required packages for Angular 2 and Webpack. The Webpack packages are all added to the devDependencies. A "npm build" script and also a "npm buildProduction" are also configured, so that the client application can be built using Webpack from the cmd line using "npm build" or "npm buildProduction". These two scripts just call the same cmd as the Webpack task runner.

{% highlight js %}
{
  "version": "1.0.0",
  "description": "",
  "main": "wwwroot/index.html",
  "author": "",
  "license": "ISC",
  "scripts": {
     "start": "webpack-dev-server --inline --progress --port 8080",
    "build": "SET NODE_ENV=development &amp;&amp; webpack -d --color",
    "buildProduction": "SET NODE_ENV=production &amp;&amp; webpack -d --color",
    "tsc": "tsc",
    "tsc:w": "tsc -w",
    "typings": "typings",
    "postinstall": "typings install"
  },
  "dependencies": {
    "@angular/common": "2.0.0-rc.5",
    "@angular/compiler": "2.0.0-rc.5",
    "@angular/core": "2.0.0-rc.5",
    "@angular/forms": "0.3.0",
    "@angular/http": "2.0.0-rc.5",
    "@angular/platform-browser": "2.0.0-rc.5",
    "@angular/platform-browser-dynamic": "2.0.0-rc.5",
    "@angular/router": "3.0.0-rc.1",
    "@angular/upgrade": "2.0.0-rc.5",
    "core-js": "^2.4.0",
    "reflect-metadata": "^0.1.3",
    "rxjs": "5.0.0-beta.6",
    "zone.js": "^0.6.12",

    "bootstrap": "^3.3.6",
        "extract-text-webpack-plugin": "^1.0.1"
  },
  "devDependencies": {
    "autoprefixer": "^6.3.2",
    "clean-webpack-plugin": "^0.1.9",
    "copy-webpack-plugin": "^2.1.3",
    "css-loader": "^0.23.0",
    "extract-text-webpack-plugin": "^1.0.1",
    "file-loader": "^0.8.4",
    "html-loader": "^0.4.0",
    "html-webpack-plugin": "^2.8.1",
    "jquery": "^2.2.0",
    "json-loader": "^0.5.3",
    "node-sass": "^3.4.2",
    "null-loader": "0.1.1",
    "postcss-loader": "^0.9.1",
    "raw-loader": "0.5.1",
    "rimraf": "^2.5.1",
    "sass-loader": "^3.1.2",
    "style-loader": "^0.13.0",
    "ts-helpers": "^1.1.1",
    "ts-loader": "0.8.2",
    "typescript": "1.8.10",
    "typings": "1.0.4",
    "url-loader": "^0.5.6",
    "webpack": "1.13.1",
    "webpack-dev-server": "^1.14.1"
  }
}
{% endhighlight %}

**typings configuration**

The typings are configured for webpack builds.

{% highlight js %}
{
    "globalDependencies": {
        "core-js": "registry:dt/core-js#0.0.0+20160602141332",
        "node": "registry:dt/node#6.0.0+20160621231320"
    }
}
{% endhighlight %}

**tsconfig configuration**

The tsconfig is configured to use commonjs as the module.

{% highlight js %}
{
    "compilerOptions": {
        "target": "es5",
        "module": "commonjs",
        "moduleResolution":  "node",
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
}
{% endhighlight %}

**Webpack build**

[https://github.com/preboot/angular2-webpack](https://github.com/preboot/angular2-webpack). Thanks for this. Small changes were made to this, such as the process.env.NODE_ENV and Webpack uses different source and output folders to match the ASP.NET Core project. If you decide to use two different projects, one for server, and one for client, preboot or angular-cli, or both together would be a good choice for the client application.

**webpack.config.js**
{% highlight js %}
/// <binding ProjectOpened='Run - Development' />

var environment = (process.env.NODE_ENV || "development").trim();

if (environment === "development") {
    module.exports = require('./webpack.dev.js');
} else {
    module.exports = require('./webpack.prod.js');
}
{% endhighlight %}

**webpack.dev.js**

{% highlight js %}
var path = require('path');
var webpack = require('webpack');

var CommonsChunkPlugin = webpack.optimize.CommonsChunkPlugin;
var Autoprefixer = require('autoprefixer');
var HtmlWebpackPlugin = require('html-webpack-plugin');
var ExtractTextPlugin = require('extract-text-webpack-plugin');
var CopyWebpackPlugin = require('copy-webpack-plugin');
var CleanWebpackPlugin = require('clean-webpack-plugin');
var helpers = require('./webpack.helpers');

module.exports = {

    debug: true,
    //watch: true,
    devtool: 'eval-source-map',

    entry: {
        'polyfills': './angular2App/polyfills.ts',
        'vendor': './angular2App/vendor.ts',
        'app': './angular2App/boot.ts' // our angular app
    },

    output: {
        path: "./wwwroot/",
        filename: 'dist/[name].bundle.js',
        publicPath: "/"
    },

    resolve: {
        extensions: ['', '.ts', '.js', '.json', '.css', '.scss', '.html']
    },

    devServer: {
        historyApiFallback: true,
        stats: 'minimal',
        outputPath: path.join(__dirname, 'wwwroot/')
    },

    module: {
        loaders: [
            {
                test: /\.ts$/,
                loader: 'ts',
                query: {
                    'ignoreDiagnostics': [
                        2403, // 2403 -> Subsequent variable declarations
                        2300, // 2300 -> Duplicate identifier
                        2374, // 2374 -> Duplicate number index signature
                        2375, // 2375 -> Duplicate string index signature
                        2502 // 2502 -> Referenced directly or indirectly
                    ]
                },
                exclude: [/node_modules\/(?!(ng2-.+))/]
            },

            // copy those assets to output
            {
                test: /\.(png|jpg|gif|ico|woff|woff2|ttf|svg|eot)$/,
                exclude: /node_modules/,
                loader: "file?name=assets/[name]-[hash:6].[ext]",
            },

            // Load css files which are required in vendor.ts
            {
                test: /\.css$/,
                exclude: /node_modules/,
                loader: "style-loader!css-loader"
            },

            {
                test: /\.scss$/,
                exclude: /node_modules/,
                loader: 'raw-loader!style-loader!css-loader!sass-loader'
            },

            {
                test: /\.html$/,
                loader: 'raw'
            }
        ],
        noParse: [/.+zone\.js\/dist\/.+/, /.+angular2\/bundles\/.+/, /angular2-polyfills\.js/]
    },

    plugins: [
        new CleanWebpackPlugin(
            [
                './wwwroot/dist',
                './wwwroot/fonts',
                './wwwroot/assets'
            ]
        ),

        new CommonsChunkPlugin({
            name: ['vendor', 'polyfills']
        }),

        new HtmlWebpackPlugin({
            filename: 'index.html',
            inject: 'body',
            chunksSortMode: helpers.packageSort(['polyfills', 'vendor', 'app']),
            template: 'angular2App/index.html'
        }),

        new CopyWebpackPlugin([
            { from: './angular2App/images/*.*', to: "assets/", flatten: true }
        ])
    ]
};
{% endhighlight %}

**webpack.prod.js**

{% highlight js %}
var path = require('path');
var webpack = require('webpack');

var CommonsChunkPlugin = webpack.optimize.CommonsChunkPlugin;
var Autoprefixer = require('autoprefixer');
var HtmlWebpackPlugin = require('html-webpack-plugin');
var ExtractTextPlugin = require('extract-text-webpack-plugin');
var CopyWebpackPlugin = require('copy-webpack-plugin');
var CleanWebpackPlugin = require('clean-webpack-plugin');
var helpers = require('./webpack.helpers');

module.exports = {

    entry: {
        'polyfills': './angular2App/polyfills.ts',
        'vendor': './angular2App/vendor.ts',
        'app': './angular2App/boot.ts' // our angular app
    },

    output: {
        path: "./wwwroot/",
        filename: 'dist/[name].[hash].bundle.js',
        publicPath: "/"
    },

    resolve: {
        extensions: ['', '.ts', '.js', '.json', '.css', '.scss', '.html']
    },

    devServer: {
        historyApiFallback: true,
        stats: 'minimal',
        outputPath: path.join(__dirname, 'wwwroot/')
    },

    module: {
        loaders: [
            {
                test: /\.ts$/,
                loader: 'ts',
                query: {
                    'ignoreDiagnostics': [
                        2403, // 2403 -> Subsequent variable declarations
                        2300, // 2300 -> Duplicate identifier
                        2374, // 2374 -> Duplicate number index signature
                        2375, // 2375 -> Duplicate string index signature
                        2502 // 2502 -> Referenced directly or indirectly
                    ]
                },
                exclude: [/node_modules\/(?!(ng2-.+))/]
            },

            // copy those assets to output
            {
                test: /\.(png|jpg|gif|ico|woff|woff2|ttf|svg|eot)$/,
                exclude: /node_modules/,
                loader: "file?name=assets/[name]-[hash:6].[ext]",
            },

            // Load css files which are required in vendor.ts
            {
                test: /\.css$/,
                exclude: /node_modules/,
                loader: "style-loader!css-loader"
            },

            {
                test: /\.scss$/,
                exclude: /node_modules/,
                loader: 'raw-loader!style-loader!css-loader!sass-loader'
            },

            {
                test: /\.html$/,
                loader: 'raw'
            }
        ],
        noParse: [/.+zone\.js\/dist\/.+/, /.+angular2\/bundles\/.+/, /angular2-polyfills\.js/]
    },

    plugins: [
        new CleanWebpackPlugin(
            [
                './wwwroot/dist',
                './wwwroot/fonts',
                './wwwroot/assets'
            ]
        ),
        new webpack.NoErrorsPlugin(),
        new webpack.optimize.DedupePlugin(),
        new webpack.optimize.UglifyJsPlugin({
            compress: {
                warnings: false
            }
        }),
        new CommonsChunkPlugin({
            name: ['vendor', 'polyfills']
        }),

        new HtmlWebpackPlugin({
            filename: 'index.html',
            inject: 'body',
            chunksSortMode: helpers.packageSort(['polyfills', 'vendor', 'app']),
            template: 'angular2App/index.html'
        }),

        new CopyWebpackPlugin([
            { from: './angular2App/images/*.*', to: "assets/", flatten: true }
        ])
    ]
};
{% endhighlight %}

**webpack.helpers.js**

{% highlight js %}
var path = require('path');

module.exports = {
    // Helper functions
    root: function (args) {
        args = Array.prototype.slice.call(arguments, 0);
        return path.join.apply(path, [__dirname].concat(args));
    },

    packageSort: function (packages) {
        // packages = ['polyfills', 'vendor', 'app']
        var len = packages.length - 1;
        var first = packages[0];
        var last = packages[len];
        return function sort(a, b) {
            // polyfills always first
            if (a.names[0] === first) {
                return -1;
            }
            // main always last
            if (a.names[0] === last) {
                return 1;
            }
            // vendor before app
            if (a.names[0] !== first &amp;&amp; b.names[0] === last) {
                return -1;
            } else {
                return 1;
            }
        };
    }
};
{% endhighlight %}

Lets dive into the webpack.dev.js a bit:

First, all plugins are loaded which are required to process all the js, ts, ... files which are included, or used in the project.

{% highlight js %}
var path = require('path');
var webpack = require('webpack');

var CommonsChunkPlugin = webpack.optimize.CommonsChunkPlugin;
var Autoprefixer = require('autoprefixer');
var HtmlWebpackPlugin = require('html-webpack-plugin');
var ExtractTextPlugin = require('extract-text-webpack-plugin');
var CopyWebpackPlugin = require('copy-webpack-plugin');
var CleanWebpackPlugin = require('clean-webpack-plugin');
var helpers = require('./webpack.helpers');
{% endhighlight %}

The npm environment variable NODE_ENV is used to define the type of build, either a development build or a production build. The entries are configured depending on this parameter.

{% highlight js %}
    config.entry = {
        'polyfills': './angular2App/polyfills.ts',
        'vendor': './angular2App/vendor.ts',
        'app': './angular2App/boot.ts' // our angular app
    };
{% endhighlight %}

The entries provide Webpack with the required information, where to start from, or where to hook in to. Three entry points are defined in this configuration. These strings point to the files required in the solution. The starting point for the app itself is provided in one of these files, boot.ts as a starting-point and also all vendor scripts minified in one file, the vendor.ts.


{% highlight js %}
import 'jquery/src/jquery';
import 'bootstrap/dist/js/bootstrap';

import 'bootstrap/dist/css/bootstrap.css';
import 'bootstrap/dist/css/bootstrap-theme.css';

import '../favicon.ico';
{% endhighlight %}

Webpack knows which paths to run and includes the corresponding files and packages.

The "loaders" section and the "modules" section in the configuration provides Webpack with the following information: which files it needs to get and how to read the files. The modules tells Webpack what to do with the files exactly. Like minifying or whatever.

In this project configuration, if a production node parameter is set, different plugins are pushed into the sections because the files should be treated differently.

**The output**

{% highlight js %}
output: {
        path: "./wwwroot/",
        filename: 'dist/[name].bundle.js',
        publicPath: "/"
    },
{% endhighlight %}

tells webpack where to put the files in the end. You can use like wildcards to use the "name" or an "hash" or something like that.

**The module loaders**

{% highlight js %}
module: {
        loaders: [
           //...loaders here
        ]
    },
{% endhighlight %}

tell webpack how to react when a certain file extension comes into play. It will then use loaders to handle that file.

The plugins you are providing in the end are necessary to configure how the files should be processed.

{% highlight js %}
plugins: [
    //...loaders here
]
{% endhighlight %}

**Angular 2 index.html**

The index.html contains all the references required for the Angular 2 client. The scripts are added as part of the build and not manually. The developer only needs to use the imports.

Source index.html file in the angular2App/public folder:

&nbsp;

{% highlight html %}
<!doctype html>
<html>
<head>
    <base href="./">
 
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Angular 2 Webpack Demo</title>
 
    <meta http-equiv="content-type" content="text/html; charset=utf-8" />
 
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
 
</head>
<body>
    <my-app>Loading...</my-app>
</body>
</html>
{% endhighlight %}

And the produced build file in the wwwroot folder. The scripts for the app, vendor and boot have been added using Webpack. Hashes are used in a production build for cache busting.

{% highlight html %}
<!doctype html>
<html>
<head>
    <base href="./">
 
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Angular 2 Webpack Demo</title>
 
    <meta http-equiv="content-type" content="text/html; charset=utf-8" />
 
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
 
    <link rel="stylesheet" href="css/bootstrap.css">
</head>
<body>
    <my-app>Loading...</my-app>
<script type="text/javascript" src="http://localhost:5000/dist/polyfills.js"></script><script type="text/javascript" src="http://localhost:5000/dist/vendor.js"></script><script type="text/javascript" src="http://localhost:5000/dist/app.js"></script></body>
</html>
{% endhighlight %}

**Visual Studio tools**

[Webpack task runner ](https://visualstudiogallery.msdn.microsoft.com/5497fd10-b1ba-474c-8991-1438ae47012a) from Mads Kristensen can be downloaded and used to send Webpack commands using the webpack.config.js file. The node NODE_ENV parameter is used to define the build type. The parameter can be set to "development", or "production".

![vs_webpack_angular2_02](http://offering.solutions/wp-content/uploads/2016/06/vs_webpack_angular2_02.png)

The Webpack task runner can also be used by double clicking the task. The execution results are then displayed in the task runner console.

![vs_webpack_angular2_03](http://offering.solutions/wp-content/uploads/2016/06/vs_webpack_angular2_03.png)

This runner provides a number of useful commands which can be activated automatically. These tasks can be attached to Visual Studio events by right clicking the task and selecting a binding. This adds a binding tag to the webpack.config.js file.

```/// <binding ProjectOpened='Run - Development' />```


**Webpack SASS**

[SASS](http://sass-lang.com/) is used to style the SPA application. The SASS files can be built using the SASS loader. Webpack can build all the styles inline or as an external file, depending on your Webpack config.

{% highlight js %}
{
  test: /\.scss$/,
  exclude: root('angular2App', 'app'),
  loader: ExtractTextPlugin.extract('style', 'css?sourceMap!postcss!sass')
},
{% endhighlight %}

**Webpack Clean**

[clean-webpack-plugin](https://github.com/johnagan/clean-webpack-plugin/) is used to clean up the deployment folder inside the wwwroot. This ensures that the application uses the latest files.

The clean task can be configured as follows:

```var CleanWebpackPlugin = require('clean-webpack-plugin');```

And used in Webpack.

```new CleanWebpackPlugin(['./wwwroot/dist']),```

**Angular 2 component files**

The Angular 2 components are slightly different to the standard example components. The templates and the styles use require, which adds the html or the css, scss to the file directly using Webpack, or as an external link depending on the Webpack config.

{% highlight ts %}
import { Observable } from 'rxjs/Observable';
import { Component, OnInit } from '@angular/core';
import { CORE_DIRECTIVES } from '@angular/common';
import { Http } from '@angular/http';
import { DataService } from '../services/DataService';
 
 
@Component({
    selector: 'homecomponent',
    template: require('./home.component.html'),
    directives: [CORE_DIRECTIVES],
    providers: [DataService]
})
 
export class HomeComponent implements OnInit {
 
    public message: string;
    public values: any[];
 
    constructor(private _dataService: DataService) {
        this.message = "Hello from HomeComponent constructor";
    }
 
    ngOnInit() {
        this._dataService
            .GetAll()
            .subscribe(data => this.values = data,
            error => console.log(error),
            () => console.log('Get all complete'));
    }
}
{% endhighlight %}

**The ASP.NET Core API**

The ASP.NET Core API is quite small and tiny. It just provides a demo CRUD service.

{% highlight cs %}
[Route("api/[controller]")]
public class ValuesController : Microsoft.AspNetCore.Mvc.Controller
{
    // GET: api/values
    [HttpGet]
    public IActionResult Get()
    {
        return new JsonResult(new string[] { "value1", "value2" });
    }

    // GET api/values/5
    [HttpGet("{id}")]
    public IActionResult Get(int id)
    {
        return new JsonResult("value");
    }

    // POST api/values
    [HttpPost]
    public IActionResult Post([FromBody]string value)
    {
        return new CreatedAtRouteResult("anyroute", null);
    }

    // PUT api/values/5
    [HttpPut("{id}")]
    public IActionResult Put(int id, [FromBody]string value)
    {
        return new OkResult();
    }

    // DELETE api/values/5
    [HttpDelete("{id}")]
    public IActionResult Delete(int id)
    {
        return new NoContentResult();
    }
}
{% endhighlight %}

**The Angular2 Http-Service**

Note that in a normal environment, you should always return the typed classes and never the plain HTTP response like here. This application only has strings to return, and this is enough for the demo.

{% highlight ts %}
import { Injectable } from '@angular/core';
import { Http, Response, Headers } from '@angular/http';
import 'rxjs/add/operator/map'
import { Observable } from 'rxjs/Observable';
import { Configuration } from '../app.constants';
 
@Injectable()
export class DataService {
 
    private actionUrl: string;
    private headers: Headers;
 
    constructor(private _http: Http, private _configuration: Configuration) {
 
        this.actionUrl = _configuration.Server + 'api/values/';
 
        this.headers = new Headers();
        this.headers.append('Content-Type', 'application/json');
        this.headers.append('Accept', 'application/json');
    }
 
    public GetAll = (): Observable =&amp;gt; {
        return this._http.get(this.actionUrl).map((response: Response) =&amp;gt; response.json());
    }
 
    public GetSingle = (id: number): Observable =&amp;gt; {
        return this._http.get(this.actionUrl + id).map(res =&amp;gt; res.json());
    }
 
    public Add = (itemName: string): Observable =&amp;gt; {
        var toAdd = JSON.stringify({ ItemName: itemName });
 
        return this._http.post(this.actionUrl, toAdd, { headers: this.headers }).map(res =&amp;gt; res.json());
    }
 
    public Update = (id: number, itemToUpdate: any): Observable =&amp;gt; {
        return this._http
            .put(this.actionUrl + id, JSON.stringify(itemToUpdate), { headers: this.headers })
            .map(res =&amp;gt; res.json());
    }
 
    public Delete = (id: number): Observable =&amp;gt; {
        return this._http.delete(this.actionUrl + id);
    }
}
{% endhighlight %}

**Notes:**

The Webpack configuration could also build all of the scss and css files to a separate app.css or app."hash".css which could be loaded as a single file in the distribution. Some of the vendor js and css could also be loaded directly in the html header using the index.html file and not included in the Webpack build.

If you are building both the client application and the server application in separate projects, you could also consider angular-cli of angular2-webpack for the client application.

Debugging the Angular 2 in Visual Studio with breakpoints is not possible with this setup. The SPA app can be debugged in chrome.

**Links:**

[https://github.com/preboot/angular2-webpack](https://github.com/preboot/angular2-webpack)

[https://webpack.github.io/docs/](https://webpack.github.io/docs/)

[https://github.com/jtangelder/sass-loader](https://github.com/jtangelder/sass-loader)

[https://github.com/petehunt/webpack-howto/blob/master/README.md](https://github.com/petehunt/webpack-howto/blob/master/README.md)

[http://www.sochix.ru/how-to-integrate-webpack-into-visual-studio-2015/](http://www.sochix.ru/how-to-integrate-webpack-into-visual-studio-2015/)

[http://sass-lang.com/](http://sass-lang.com/)

[WebPack Task Runner ](https://visualstudiogallery.msdn.microsoft.com/5497fd10-b1ba-474c-8991-1438ae47012a) from Mads Kristensen

[http://blog.thoughtram.io/angular/2016/06/08/component-relative-paths-in-angular-2.html](http://blog.thoughtram.io/angular/2016/06/08/component-relative-paths-in-angular-2.html)

[https://angular.io/docs/ts/latest/guide/webpack.html](https://angular.io/docs/ts/latest/guide/webpack.html)
