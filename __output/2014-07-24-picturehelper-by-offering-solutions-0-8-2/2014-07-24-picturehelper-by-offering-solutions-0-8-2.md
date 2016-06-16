---
id: 297
title: '[NuGet] &#8211; PictureHelper by Offering.Solutions 0.8.2'
date: 2014-07-24T20:12:54+00:00
author: Fabian Gosebrink
layout: post
tags: function helper nuget picture 
logo: 'assets/images/logo_small.png'
navigation: True
cover: 'assets/images/download_edit_dark.jpg'
subclass: 'post tag-speeches'
---

After dealing a bit with pictures in web-applications I decided to put the functionalities I used into a service. Then after using it again and again I decided to make a NuGet-Package out of it so that I can reuse it easier. Well, I got also tired of copy/paste 😉 And now I want to share this with you. Perhaps anybody else needs to have small basic picture functions, here you are:

<div class="package-page-heading">
  <h3>
    <a href="https://www.nuget.org/packages/OfferingSolutions.PictureHelper/0.8.2">PictureHelper by Offering.Solutions 0.8.2</a>
  </h3>
  
  <p>
    &nbsp;
  </p>
  
  <p>
    Installing is very easy: Just copy/paste or type the line showed into you Package Manager Console in Visual Studio. Its just one dll and really, its very small. Its my first try for NugetPackages though. (Perhaps you will see more the next time 😉 )
  </p>
  
  <p>
    <a href="{{site.baseurl}}assets/images/blogs/2014-07/2b9208f3-0928-4471-9556-1deca17215cd.jpg"><img class="aligncenter size-full wp-image-303" src="{{site.baseurl}}assets/images/blogs/2014-07/2b9208f3-0928-4471-9556-1deca17215cd.jpg" alt="2" width="761" height="117" srcset="{{site.baseurl}}assets/images/blogs/2014-07/2b9208f3-0928-4471-9556-1deca17215cd.jpg 761w, http://offering.solutions/wp-content/uploads/2014/07/21-300x46.jpg 300w" sizes="(max-width: 761px) 100vw, 761px" /></a>
  </p>
  
  <p>
    Basic functions are:
  </p>
  
  <ul>
    <li>
      Saving an image with different options
    </li>
    <li>
      Changing opacity of an image (Png/Gif)
    </li>
    <li>
      Resizing an image
    </li>
    <li>
      Crop an image
    </li>
  </ul>
</div>

<div class="package-page-heading">
  Which can look like this:
</div>

&nbsp;

<pre class="lang:c# decode:true">static void Main(string[] args)
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
        }</pre>

I hope these functions are understandable and easy to use. If you have any improvements: Let me know! This is a beta status, the release number is set to 0.8.2. But all functions are manually tested and also covered with automatic unit-tests in a continuous integration environment.

&nbsp;

Sample solution is here:

[Offering.Solutions.PictureHelperSample](http://offering.solutions/wp-content/uploads/2014/07/Offering.Solutions.PictureHelperSample.zip)

&nbsp;

Hope you enjoy it.
  
Regards

&nbsp;

Fabian
