---
id: 167
title: 'WPF crashes with "OutOfMemoryException" when loading PNGs in Windows 8.1'
date: 2014-06-04T16:05:34+00:00
author: Fabian Gosebrink
layout: post
tags: exception images outofmemory pictures windows8.1 WPF 
logo: 'assets/images/logo_small.png'
navigation: True
cover: 'assets/images/aerial-view-of-laptop-and-notebook_bw_osc.jpg'
subclass: 'post tag-speeches'
disqus: true
categories: blog articles
---

In this blogpost I want to show you the solution for an error I had today which I spent a lot of time on solving it. it was the case that WPF crashes with OutOfMemoryException when loading PNGs.

I had Windows 8.1 running as operating system and used pngs in my WPF application. It was all running well until I wanted to display the images. The whole programm froze and was unusable. Had to kill the process. I spent a lot of time on this and went through a lot of possibilities to display images in WPF.

It turned out that this is because of indexed and not indexed pngs (You can google the difference). WPF in windows 8.1 can not handle indexed png-files. Open them, save them with RGB (e.g. with Photoshop) and you are done.

You can do Unit Tests with the following function: <a href="http://msdn.microsoft.com/en-us/library/system.drawing.graphics.fromimage.aspx">Graphics.FromImage()</a>

>"If the image has an indexed pixel format, this method throws an exception with the message, "A Graphics object cannot be created from an image that has an indexed pixel format.""

So load all your images with "Graphic.ImageFrom(...)" and see if the exception is thrown to test this.

Hope this helps.

Regards

Fabian
