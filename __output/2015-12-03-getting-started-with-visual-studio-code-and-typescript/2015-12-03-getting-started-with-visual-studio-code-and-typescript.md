---
id: 833
title: Getting started with Visual Studio Code and Typescript
date: 2015-12-03T14:17:33+00:00
author: Fabian Gosebrink
layout: post
tags: angularjs typescript visualstudiocode 
logo: 'assets/images/logo_small.png'
navigation: True
cover: 'assets/images/download_edit_dark.jpg'
subclass: 'post tag-speeches'
---

## Getting started with Visual Studio Code and Typescript

Hey,

this time I want to show you how to get started with Visual Studio Code and Typescript.

Before we start: A new Typescript version was announced!! You can see whats new [## Getting started with Visual Studio Code and Typescript

Hey,

this time I want to show you how to get started with Visual Studio Code and Typescript.

Before we start: A new Typescript version was announced!! You can see whats new](https://github.com/Microsoft/TypeScript/wiki/roadmap) and further information [## Getting started with Visual Studio Code and Typescript

Hey,

this time I want to show you how to get started with Visual Studio Code and Typescript.

Before we start: A new Typescript version was announced!! You can see whats new [## Getting started with Visual Studio Code and Typescript

Hey,

this time I want to show you how to get started with Visual Studio Code and Typescript.

Before we start: A new Typescript version was announced!! You can see whats new](https://github.com/Microsoft/TypeScript/wiki/roadmap) and further information](http://www.typescriptlang.org/) 

I always wanted to get to know Typescript and see the advantages of it compared to the &#8220;normal&#8221; javascript and so I just took my plain old sample application and tried to build it up with Typescript. I have to say: Typescript in combination with Visual Studio Code is absolutely awesome! I loved it.

## Getting Started:

Fist of all you need to install npm/nodejs from the installers [## Getting started with Visual Studio Code and Typescript

Hey,

this time I want to show you how to get started with Visual Studio Code and Typescript.

Before we start: A new Typescript version was announced!! You can see whats new [## Getting started with Visual Studio Code and Typescript

Hey,

this time I want to show you how to get started with Visual Studio Code and Typescript.

Before we start: A new Typescript version was announced!! You can see whats new](https://github.com/Microsoft/TypeScript/wiki/roadmap) and further information [## Getting started with Visual Studio Code and Typescript

Hey,

this time I want to show you how to get started with Visual Studio Code and Typescript.

Before we start: A new Typescript version was announced!! You can see whats new [## Getting started with Visual Studio Code and Typescript

Hey,

this time I want to show you how to get started with Visual Studio Code and Typescript.

Before we start: A new Typescript version was announced!! You can see whats new](https://github.com/Microsoft/TypeScript/wiki/roadmap) and further information](http://www.typescriptlang.org/) 

I always wanted to get to know Typescript and see the advantages of it compared to the &#8220;normal&#8221; javascript and so I just took my plain old sample application and tried to build it up with Typescript. I have to say: Typescript in combination with Visual Studio Code is absolutely awesome! I loved it.

## Getting Started:

Fist of all you need to install npm/nodejs from the installers](https://nodejs.org/en/) if you are running a windows machine.

if you have npm installed you can install typescript with the command

> <pre class="">npm install -g typescript</pre>

which will install typescript globally. With this installed you can run the typescript compiler &#8220;tsc&#8221; and compile typescript.

With

> <pre class="">npm install tsd -g</pre>

You can install the [TypeScript Definition manager for DefinitelyTyped](http://definitelytyped.org/tsd/) to get the strong typed advantages from Typescript going.

In Visual Studio Code the first file you need when working with Typescript is the &#8220;tsconfig.json&#8221; which sets the basic behaviour of your typescript compiler.

<pre class="lang:js decode:true ">{
	"compilerOptions": {
		"target": "ES5"
	},
	"exclude": [
        "node_modules",
        "wwwroot"
    ]
}</pre>

The &#8220;compilerOptions&#8221; tell typescript in which language the typescript code should be compiled. The exclude-block, you guessed it, excluded several folders from being checked by the compiler.

Now you can write and compile Typescript files. 🙂

In Visual Studio Code if you now press the CRTL-SHIFT-B-Buttons you should trigger Visual Studio Code to compile your written files. If you create a taskrunner Visual Studio Code generates a .vscode-folder with a &#8220;tasks.json&#8221; in it.

![alttext]({{site.baseurl}}assets/images/blogs/2015-12/fba19471-7e44-4025-8a81-8214275c2e6e.jpg)See that the &#8220;args&#8221; is not containing any files explicitly. That triggers the taskrunner to compile the whole code. Not only several files.

&nbsp;

If you now type Typescript-code in a *.ts file, save it and compile you will see a progress running in vs code for a short time. After this your file should be compiled into Typescript and appear in Visual Studio Code since Visual Studio Code is file based.

![alttext]({{site.baseurl}}assets/images/blogs/2015-12/18205430-f3c5-4907-b54f-6519c12b6fc8.jpg) ![alttext](http://offering.solutions/wp-content/uploads/2015/12/tyepscript03.jpg)

> If you do NOT see any compiled js/ts-files checking your typescript version is always a good hint.

### Adding AngularJs-Files

If you want to get also started with AngularJs (and have installed the tsd like mentioned above) you can go to the root of your application, open the console and simply type

> <pre class="">tsd install angular --resolve --save</pre>

which causes the Typescript Definition Manager (tsd) to install the *.d.ts files for AngularJs. The resolve parameter resolves all dependencies angular has (like jQuery in this case).

![alttext]({{site.baseurl}}assets/images/blogs/2015-12/3a833894-0569-4de1-909c-2a68f315018b.jpg)

As you can see the typings-folder was generated with a &#8220;tsd.d.ts&#8221; file which holds the references and a tsd.json on the root of the application which holds all installed plugins for our project. This gets updated every time you install a new typescript.d-reference.

&nbsp;

Thats it. You can now go ahead and implement Typescript with Visual Studio Code.

Check out the github repository of this code [here](https://github.com/FabianGosebrink/AspNetWebApiAngularWithTypescript).

&nbsp;

Hope this helps.

&nbsp;

Regards

Fabian