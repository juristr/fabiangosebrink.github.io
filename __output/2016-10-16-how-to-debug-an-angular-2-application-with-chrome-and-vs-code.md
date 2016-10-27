---
id: 1749
title: How to debug an Angular 2 application with Chrome and VS Code
date: 2016-10-16T17:50:43+00:00
author: Fabian Gosebrink
layout: post
guid: https://offering.solutions/?p=1749
permalink: /articles/angular/how-to-debug-an-angular-2-application-with-chrome-and-vs-code/
dsq_thread_id:
  - 5227794679
categories:
  - Angular
tags:
  - Angular2
  - Chrome
  - VSCode
---
In this blogpost I want to show you how to debug an Angular 2 application with Chrome and VS Code.

First of all you need to install the extension in VS Code.

You can find it here

<https://github.com/Microsoft/vscode-chrome-debug>

or search in the extensions tab for the plugin directly:

[<img class="aligncenter wp-image-1751 size-full" src="https://offering.solutions/wp-content/uploads/2016/10/HowtodebuganAngular2applicationwithChromeandVSCode_01.jpg" alt="How to debug an Angular 2 application with Chrome and VS Code" width="400" height="289" srcset="https://offering.solutions/wp-content/uploads/2016/10/HowtodebuganAngular2applicationwithChromeandVSCode_01.jpg 400w, https://offering.solutions/wp-content/uploads/2016/10/HowtodebuganAngular2applicationwithChromeandVSCode_01-300x217.jpg 300w" sizes="(max-width: 400px) 100vw, 400px" />](https://offering.solutions/wp-content/uploads/2016/10/HowtodebuganAngular2applicationwithChromeandVSCode_01.jpg)After installing you probably have to enable the plugin and restart VS Code but in the end you will see your folder structure like normal. Then head over to the debug tab and press the button for creating you an new configuration and select the "Chrome" environment.

[<img class="aligncenter size-large wp-image-1752" src="https://offering.solutions/wp-content/uploads/2016/10/HowtodebuganAngular2applicationwithChromeandVSCode_02-1024x276.jpg" alt="howtodebuganangular2applicationwithchromeandvscode_02" width="750" height="202" srcset="https://offering.solutions/wp-content/uploads/2016/10/HowtodebuganAngular2applicationwithChromeandVSCode_02-1024x276.jpg 1024w, https://offering.solutions/wp-content/uploads/2016/10/HowtodebuganAngular2applicationwithChromeandVSCode_02-300x81.jpg 300w, https://offering.solutions/wp-content/uploads/2016/10/HowtodebuganAngular2applicationwithChromeandVSCode_02-768x207.jpg 768w, https://offering.solutions/wp-content/uploads/2016/10/HowtodebuganAngular2applicationwithChromeandVSCode_02.jpg 1252w" sizes="(max-width: 750px) 100vw, 750px" />](https://offering.solutions/wp-content/uploads/2016/10/HowtodebuganAngular2applicationwithChromeandVSCode_02.jpg)After doing this the extension created a new folder (if you do not have it already) called ".vscode" and a "launch.json" in it initially looking like this:

&nbsp;

<pre class="lang:js decode:true ">{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Launch Chrome against localhost, with sourcemaps",
            "type": "chrome",
            "request": "launch",
            "url": "http://localhost:8080",
            "sourceMaps": true,
            "webRoot": "${workspaceRoot}"
        },
        {
            "name": "Attach to Chrome, with sourcemaps",
            "type": "chrome",
            "request": "attach",
            "port": 9222,
            "sourceMaps": true,
            "webRoot": "${workspaceRoot}"
        }
    ]
}</pre>

Our folder strucutre tells us that the files are served from the root.

[<img class="aligncenter size-full wp-image-1753" src="https://offering.solutions/wp-content/uploads/2016/10/HowtodebuganAngular2applicationwithChromeandVSCode_03.jpg" alt="howtodebuganangular2applicationwithchromeandvscode_03" width="382" height="450" srcset="https://offering.solutions/wp-content/uploads/2016/10/HowtodebuganAngular2applicationwithChromeandVSCode_03.jpg 382w, https://offering.solutions/wp-content/uploads/2016/10/HowtodebuganAngular2applicationwithChromeandVSCode_03-255x300.jpg 255w" sizes="(max-width: 382px) 100vw, 382px" />](https://offering.solutions/wp-content/uploads/2016/10/HowtodebuganAngular2applicationwithChromeandVSCode_03.jpg)So the ""webRoot": "${workspaceRoot}"" -setting is good to go for us. We will open a new Chrome instance but it needs an existing running server. So if you use something like "[lite-server](https://github.com/johnpapa/lite-server)" you can easily type "lite-server" at the root of your webapplication or place it in your npm command chain in the "npm start" command. This is what I did. But before we go we need to sdjust the urls where the server is running on and the url where the Chrome instance is starting.

So replace the port in the config file with the port from your lite-server. In my cae thats "3000". This is how your config look like then:

<pre class="lang:js decode:true ">{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Launch Chrome against localhost, with sourcemaps",
            "type": "chrome",
            "request": "launch",
            "url": "http://localhost:3000",
            "sourceMaps": true,
            "webRoot": "${workspaceRoot}"
        },
        {
            "name": "Attach to Chrome, with sourcemaps",
            "type": "chrome",
            "request": "attach",
            "port": 9222,
            "sourceMaps": true,
            "webRoot": "${workspaceRoot}"
        }
    ]
}</pre>

Then start the lite server and just hit "play"

[<img class="aligncenter size-full wp-image-1754" src="https://offering.solutions/wp-content/uploads/2016/10/HowtodebuganAngular2applicationwithChromeandVSCode_04.jpg" alt="howtodebuganangular2applicationwithchromeandvscode_04" width="915" height="474" srcset="https://offering.solutions/wp-content/uploads/2016/10/HowtodebuganAngular2applicationwithChromeandVSCode_04.jpg 915w, https://offering.solutions/wp-content/uploads/2016/10/HowtodebuganAngular2applicationwithChromeandVSCode_04-300x155.jpg 300w, https://offering.solutions/wp-content/uploads/2016/10/HowtodebuganAngular2applicationwithChromeandVSCode_04-768x398.jpg 768w" sizes="(max-width: 915px) 100vw, 915px" />](https://offering.solutions/wp-content/uploads/2016/10/HowtodebuganAngular2applicationwithChromeandVSCode_04.jpg)

[<img class="aligncenter size-large wp-image-1755" src="https://offering.solutions/wp-content/uploads/2016/10/HowtodebuganAngular2applicationwithChromeandVSCode-1024x608.gif" alt="howtodebuganangular2applicationwithchromeandvscode" width="750" height="445" srcset="https://offering.solutions/wp-content/uploads/2016/10/HowtodebuganAngular2applicationwithChromeandVSCode-1024x608.gif 1024w, https://offering.solutions/wp-content/uploads/2016/10/HowtodebuganAngular2applicationwithChromeandVSCode-300x178.gif 300w, https://offering.solutions/wp-content/uploads/2016/10/HowtodebuganAngular2applicationwithChromeandVSCode-768x456.gif 768w" sizes="(max-width: 750px) 100vw, 750px" />](https://offering.solutions/wp-content/uploads/2016/10/HowtodebuganAngular2applicationwithChromeandVSCode.gif)

Chrome starts and you can debug your page in VS Code. Of course you can also confugre Chrome to attach directly. See here for examples:

<https://github.com/Microsoft/vscode-chrome-debug/wiki/Examples>

Hope this helps anybody

BR

Fabian