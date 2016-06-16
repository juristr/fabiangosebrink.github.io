---
id: 167
title: 'Wpf crashes with &#8220;OutOfMemoryException&#8221; when loading PNGs in Windows 8.1'
date: 2014-06-04T16:05:34+00:00
author: Fabian Gosebrink
layout: post
tags: exception images outofmemory pictures windows8.1 wpf 
logo: 'assets/images/logo_small.png'
navigation: True
cover: 'assets/images/download_edit_dark.jpg'
subclass: 'post tag-speeches'
---

Today I got the error that wpf  crashes with an OutOfMemoryException while loading Pngs. It turned out that this is because of indexed and not indexed pngs (You can google the difference). Wpf in windows 8.1 can not handle indexed png-files. Open them, save them with RGB (e.g. with Photoshop) and you are done.

You can do Unit Tests with the following function: [Graphics.FromImage()](http://msdn.microsoft.com/en-us/library/system.drawing.graphics.fromimage.aspx)

&#8220;If the image has an indexed pixel format, this method throws an exception with the message, &#8220;A Graphics object cannot be created from an image that has an indexed pixel format.&#8221;&#8221;

So load all your images with &#8220;Graphic.ImageFrom(&#8230;)&#8221; and see if the exception is thrown to test this.

Hope this helps.

Regards

Fabian
