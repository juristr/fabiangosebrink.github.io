---
id: 297
title: '[NuGet] PictureHelper by Offering.Solutions 0.8.2'
date: 2014-07-24T20:12:54+00:00
author: Fabian Gosebrink
layout: post
tags: function helper nuget picture 
logo: 'assets/images/logo_small.png'
navigation: True
cover: 'assets/images/download_edit_dark.jpg'
subclass: 'post tag-speeches'
disqus: true
categories: articles
---

After dealing a bit with pictures in web-applications I decided to put the functionalities I used into a service. Then after using it again and again I decided to make a NuGet-Package out of it so that I can reuse it easier. Well, I got also tired of copy/paste 😉 And now I want to share this with you. Perhaps anybody else needs to have small basic picture functions, here you are:

[PictureHelper by Offering.Solutions 0.8.2](https://www.nuget.org/packages/OfferingSolutions.PictureHelper/0.8.2)
 
Installing is very easy: Just copy/paste or type the line showed into you Package Manager Console in Visual Studio. Its just one dll and really, its very small. Its my first try for NugetPackages though. (Perhaps you will see more the next time 😉 )

![PictureHelper by Offering.Solutions 0.8.2]({{site.baseurl}}assets/articles/2014-07-24/edf98c8e-0f63-428a-a928-26c1d1b247f6.jpg)

Basic functions are:
* Saving an image with different options
* Changing opacity of an image (Png/Gif)
* Resizing an image
* Crop an image

Which can look like this:

```
static void Main(string[] args)
        {
            string directoryName = Path.GetDirectoryName(Assembly.GetExecutingAssembly().Location);
            string pathToImage = Path.Combine(directoryName, "BasicPicture.png");
            Image image = Image.FromFile(pathToImage);

            IOsPictureHelper osPictureHelper = new OsPictureHelper();

            // Changes the opacity of an image
            Image changeOpacity = osPictureHelper.ChangeOpacity(image, 50);

            // Crops the image with a given rectangle
            Image cropImage = osPictureHelper.CropImage(image, new Rectangle(20, 20, 100, 100));

            // Resizes the image using Size by keeping the ratio
            Image resizeImageWithSize = osPictureHelper.ResizeImage(image, new Size(50, 50));

            // Resizes the image using height and width by keeping the ratio
            Image resizeImageWithValues = osPictureHelper.ResizeImage(image, 50, 50);

            // Saves the image giving a guid as filename, returning the filename
            osPictureHelper.SaveImage(changeOpacity, directoryName);

            // Saves the image with the given filename
            osPictureHelper.SaveImage(cropImage, directoryName, "cropImage.png");

            // Saves the image with the given filename, converting it to jpg, returning the new filename
            osPictureHelper.SaveImage(resizeImageWithSize, directoryName, "resizeImageWithSize.png", ImageFormat.Jpeg);

            Console.ReadLine();
        }
        ```

I hope these functions are understandable and easy to use. If you have any improvements: Let me know! This is a beta status, the release number is set to 0.8.2. But all functions are manually tested and also covered with automatic unit-tests in a continuous integration environment.



Sample solution is here:

[Offering.Solutions.PictureHelperSample]({{site.baseurl}}assets/articles/2014-07-24/Offering.Solutions.PictureHelperSample.zip)

Hope you enjoy it.
  
Regards

Fabian
