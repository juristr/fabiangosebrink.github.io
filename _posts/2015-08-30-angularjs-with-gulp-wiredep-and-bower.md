---
id: 998
title: AngularJs with gulp, wiredep and bower
date: 2015-08-30T11:18:45+00:00
author: Fabian Gosebrink
layout: post
tags: AngularJs, bower, Development, Github, Gulp, wiredep
logo: 'assets/images/logo_small.png'
navigation: True
cover: 'assets/images/download_edit_dark.jpg'
subclass: 'post tag-speeches'
disqus: true
categories: articles
---

With this blogpost I want to show you how to use AngularJs with gulp, wiredep and bower to provide the files to an angular application.

I just introduced Gulp in my AngularJsDemoApp on GitHub.

Check it out <a href="https://github.com/FabianGosebrink/AngularJsDemoApp">here</a>.

<h3>AngularJs with gulp wiredep and bower</h3>

<h4>gulp.js</h4>

<pre class="lang:js decode:true ">var gulp = require("gulp");
var jshint = require("gulp-jshint");
var jscs = require("gulp-jscs");
var del = require("del");
var inject = require('gulp-inject');

var config = require("./gulp.config")();

gulp.task("vet", function() {
    return gulp.src(config.srcJSFiles)
        .pipe(jscs())
        .pipe(jshint())
        .pipe(jshint.reporter("jshint-stylish", { verbose: true }));
});

gulp.task('injectJsIntoIndex', ["vet"], function () {
    var wiredep = require('wiredep').stream;
    var options = config.getWiredepDefaultOptions();

    var target = gulp.src(config.targetIndexHtmlFile);
    var sources = gulp.src(config.srcJSFiles);

    return target
        .pipe(inject(sources, {
            addRootSlash: false
        }))
        .pipe(wiredep(options))
        .pipe(gulp.dest(config.root));
});</pre>

The gulp task is checking via wiredep all dependencies and is injecting them into the index.html automatically. After this it is passing out the file into the destination.

<h4>gulp.config.js</h4>

The gulp-config is seperated:

<pre class="lang:js decode:true ">module.exports = function () {

    var config = {
        srcJSFiles: [
            "./app/*.js",
            "./app/*/*.js",
            "./app/*/*/*.js",
            "!./node_modules/**/*.js"
        ],
        targetIndexHtmlFile: "index.html",
        root: "./",

        bower: {
            json: require("./bower.json"),
            directory: "./libs",
            ignorePath: "../.."
        }
    }

    config.getWiredepDefaultOptions = function () {

        var options = {
            bowerJson: config.bower.json,
            directory: config.bower.directory,
            ignorePath: config.bower.ignorePath
        };

        return options;
    };

    return config;
};</pre>

Just run the corresponding gulp commands to start the default gulp task. Or, if you do not like gulp, just comment in all the files explicitly. That will do it, too :)

<pre class="lang:xhtml decode:true "> &lt;!-- bower:js --&gt;
    &lt;script src="libs/angular/angular.js"&gt;&lt;/script&gt;
    &lt;script src="libs/angular-animate/angular-animate.js"&gt;&lt;/script&gt;
    &lt;script src="libs/angular-bootstrap/ui-bootstrap-tpls.js"&gt;&lt;/script&gt;
    &lt;script src="libs/angular-loading-bar/build/loading-bar.js"&gt;&lt;/script&gt;
    &lt;script src="libs/angular-resource/angular-resource.js"&gt;&lt;/script&gt;
    &lt;script src="libs/angular-route/angular-route.js"&gt;&lt;/script&gt;
    &lt;script src="libs/angular-toastr/dist/angular-toastr.tpls.js"&gt;&lt;/script&gt;
    &lt;script src="libs/jquery/dist/jquery.js"&gt;&lt;/script&gt;
    &lt;script src="libs/bootstrap/dist/js/bootstrap.js"&gt;&lt;/script&gt;
    &lt;script src="libs/lodash/lodash.js"&gt;&lt;/script&gt;
    &lt;!-- endbower --&gt;

    &lt;!-- inject:js --&gt;
    &lt;script src="app/application.js"&gt;&lt;/script&gt;
    &lt;script src="app/Contact/contactModule.js"&gt;&lt;/script&gt;
    &lt;script src="app/Home/homeModule.js"&gt;&lt;/script&gt;
    &lt;script src="app/Contact/Controllers/contactController.js"&gt;&lt;/script&gt;
    &lt;script src="app/Home/Controllers/homeController.js"&gt;&lt;/script&gt;
    &lt;script src="app/Home/Services/peopleServices.js"&gt;&lt;/script&gt;
    &lt;!-- endinject --&gt;</pre>

Regards and have fun. Happy coding

If you have VS 2013 you can install the <a href="https://visualstudiogallery.msdn.microsoft.com/8e1b4368-4afb-467a-bc13-9650572db708">Task Runner Explorer</a>, which is included in VS 2015 or you just run the commands from the command line being on the level of the gulp.js-file.

Fabian
