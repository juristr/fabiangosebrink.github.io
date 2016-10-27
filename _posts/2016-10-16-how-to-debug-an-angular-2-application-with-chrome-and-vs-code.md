---
id: 1749
title: How to debug an Angular 2 application with Chrome and VS Code
date: 2016-10-16T17:50:43+00:00
author: Fabian Gosebrink
layout: post
tags: Angular2 Chrome VSCode
logo: 'assets/images/logo_small.png'
navigation: True
cover: 'assets/images/download_edit_dark.jpg'
subclass: 'post tag-speeches'
disqus: true
categories: articles
---

In this blogpost I want to show you how to debug an Angular 2 application with Chrome and VS Code.

First of all you need to install the extension in VS Code.

You can find it here

<https://github.com/Microsoft/vscode-chrome-debug>

or search in the extensions tab for the plugin directly:

![How to debug an Angular 2 application with Chrome and VS Code]({{site.baseurl}}assets/articles/2016-10-164/HowtodebuganAngular2applicationwithChromeandVSCode_01.jpg)

After installing you probably have to enable the plugin and restart VS Code but in the end you will see your folder structure like normal. Then head over to the debug tab and press the button for creating you an new configuration and select the "Chrome" environment.

![How to debug an Angular 2 application with Chrome and VS Code]({{site.baseurl}}assets/articles/2016-10-164/HowtodebuganAngular2applicationwithChromeandVSCode_02.jpg)

After doing this the extension created a new folder (if you do not have it already) called ".vscode" and a "launch.json" in it initially looking like this:

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

![How to debug an Angular 2 application with Chrome and VS Code]({{site.baseurl}}assets/articles/2016-10-164/HowtodebuganAngular2applicationwithChromeandVSCode_03.jpg)

So the ""webRoot": "${workspaceRoot}"" -setting is good to go for us. We will open a new Chrome instance but it needs an existing running server. So if you use something like "[lite-server](https://github.com/johnpapa/lite-server)" you can easily type "lite-server" at the root of your webapplication or place it in your npm command chain in the "npm start" command. This is what I did. But before we go we need to sdjust the urls where the server is running on and the url where the Chrome instance is starting.

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

![How to debug an Angular 2 application with Chrome and VS Code]({{site.baseurl}}assets/articles/2016-10-164/HowtodebuganAngular2applicationwithChromeandVSCode_04.jpg)

![How to debug an Angular 2 application with Chrome and VS Code]({{site.baseurl}}assets/articles/2016-10-164/HowtodebuganAngular2applicationwithChromeandVSCode.gif))

Chrome starts and you can debug your page in VS Code. Of course you can also confugre Chrome to attach directly. See here for examples:

<https://github.com/Microsoft/vscode-chrome-debug/wiki/Examples>

Hope this helps anybody

BR

Fabian