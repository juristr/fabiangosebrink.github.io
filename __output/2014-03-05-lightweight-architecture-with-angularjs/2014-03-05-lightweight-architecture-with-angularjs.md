---
id: 65
title: Lightweight architecture with AngularJS
date: 2014-03-05T17:17:51+00:00
author: Fabian Gosebrink
layout: post
tags: angularjs javascript webarchitecture 
logo: 'assets/images/logo_small.png'
navigation: True
cover: 'assets/images/download_edit_dark.jpg'
subclass: 'post tag-speeches'
---

During the last past days I have been to a conference in Germany and brought back several interesting things I want to share with you this time. (Here I have to say that my hotel did not have a spa. So everybody who wanted to get the latest hotel and spa tips is wrong here&#8230;so in case stop reading)

<!--more-->

Well for me as a software developer it gets interesting every time I hear another company talking about their way of doing work, their kind of projects and their behavior in the market. It’s always refreshing to see how they behave. And, indeed, personally, if the topic &#8220;ASP.NET&#8221;, &#8220;Internet&#8221; with all its Buzzwords is connected you can be sure to have my attention.

This time I laid my focus on everything which was connected to the web and in eight hours of workshop and a lot of tracks the other day I learned a lot!

&nbsp;

**History**

Let me get historical for a moment: .Net in its first version was released in 2002. This is 12 years ago! 12 years are more than a lifetime in computer-science. So you feel that there is something moving. Something growing. Internet-Applications are getting bigger and bigger and: It’s getting a lot more mobile focus. Having the same application on smartphone, tablet and your desktop-PC (Do you still have one? 😉 ) is something like essential to everyone. It also solves a lot of business-issues as I heard on the conference. &#8220;Oh if we would have known that before. It would have saved us a lot of trouble!&#8221;

&nbsp;

**Why SPAs?**

ASP.NET gives us great application-possibilities for this to achieve. ASP.NET MVC gives us a very established Framework (MVC) which does a great separation of your view, your logic and dealing with and between them. But it’s a server-sided technology. You are feeling it: There is a server I have to ask every time. I hated to ask my parents for almost everything I did when I was young, why should I love to ask a Server everything I do? And, which is the bigger point, why am I feeling it while working with my application?

Wouldn&#8217;t it be nice to have an application browser-based, which looks the same on every device I have, which keeps my data synchronized no matter which device I take to change the information? I can start it on my smartphone, have it on the desktop and love the information AND the look and feel. Because I am used to it.

As I mentioned ASP.NET MVC gives us a great but server-sided-technology to build up web applications. ASP.NET WebAPI gives us only a small thing server sided: An API we can ask information from and push information to. The client-side programming language would be JavaScript. When you read &#8220;JavaScript&#8221; the first thing which comes to your mind is &#8220;jQuery&#8221;. Why? Because jQuery is one of the most common DOM-Changing tools out there in the web. Okay okay, it’s more than a DOM-Changing-Tool I know. But without Require.js and a lot of tools it’s hard to get a real structure in your application by using jQuery.

**Angular.js**

Frameworks like Angular.js are nowadays able to build up a complete MV*-Framework completely down on your client. Everybody who thought that JavaScript is a language without any structure, namespaces etc. can with tools like angular see, that this is not true. Angular.Js is a google-pushed Framework (that’s a reason why MS is not providing it in its templates 😉 ) where code can be separated into your well known controllers, into services and, of course, into your view model for html-views. The Dependency injection comes native with angular.js! With that you are facing real client-side architecture. It’s an architecture to write client-side-applications. Not only websites.

<p style="text-align: center;">
  <a href="{{site.baseurl}}assets/images/blogs/2014-03/5ab97616-0c2c-4953-9bca-8f5c20415c45.png" rel="attachment wp-att-1147"><img class="aligncenter size-full wp-image-1147" src="{{site.baseurl}}assets/images/blogs/2014-03/5ab97616-0c2c-4953-9bca-8f5c20415c45.png" alt="1" width="278" height="246" /></a>
</p>

The following screenshots shows an example of a view iterating over persons, which are called from a WebApi over services (see the following screenshots).

<p style="text-align: center;">
  <a href="{{site.baseurl}}assets/images/blogs/2014-03/6e381312-c0bc-4e45-a32a-24aef9b93832.png" rel="attachment wp-att-1148"><img class="aligncenter size-full wp-image-1148" src="{{site.baseurl}}assets/images/blogs/2014-03/6e381312-c0bc-4e45-a32a-24aef9b93832.png" alt="2" width="459" height="349" srcset="{{site.baseurl}}assets/images/blogs/2014-03/6e381312-c0bc-4e45-a32a-24aef9b93832.png 459w, http://offering.solutions/wp-content/uploads/2014/03/2-300x228.png 300w" sizes="(max-width: 459px) 100vw, 459px" /></a>
</p>

The controller gets the “scope” injected, which is an angular.js-word for representing the view model. Here the persons are called from the service and set in the corresponding property.

<a href="{{site.baseurl}}assets/images/blogs/2014-03/0232b6ae-f926-438b-8274-4ceb77a40e90.png" rel="attachment wp-att-1149"><img class="aligncenter size-full wp-image-1149" src="{{site.baseurl}}assets/images/blogs/2014-03/0232b6ae-f926-438b-8274-4ceb77a40e90.png" alt="3" width="578" height="114" srcset="{{site.baseurl}}assets/images/blogs/2014-03/0232b6ae-f926-438b-8274-4ceb77a40e90.png 578w, http://offering.solutions/wp-content/uploads/2014/03/3-300x59.png 300w" sizes="(max-width: 578px) 100vw, 578px" /></a>

Here the services for the communication to the WebAPi are shown. I only divided the service which gets injected to the controller from the one who really gets the data. Theoretically this could be one single service.

<a href="{{site.baseurl}}assets/images/blogs/2014-03/bbd2464e-0e2b-44c4-9612-26da88f05cf9.png" rel="attachment wp-att-1152"><img class="aligncenter size-full wp-image-1152" src="{{site.baseurl}}assets/images/blogs/2014-03/bbd2464e-0e2b-44c4-9612-26da88f05cf9.png" alt="41" width="562" height="443" srcset="{{site.baseurl}}assets/images/blogs/2014-03/bbd2464e-0e2b-44c4-9612-26da88f05cf9.png 562w, http://offering.solutions/wp-content/uploads/2014/03/41-300x236.png 300w" sizes="(max-width: 562px) 100vw, 562px" /></a>

Your corresponding WebApi could look like:

<a href="{{site.baseurl}}assets/images/blogs/2014-03/300d88af-838d-46e6-b4e1-a5f44f57c715.png" rel="attachment wp-att-1150"><img class="aligncenter size-full wp-image-1150" src="{{site.baseurl}}assets/images/blogs/2014-03/300d88af-838d-46e6-b4e1-a5f44f57c715.png" alt="5" width="484" height="267" srcset="{{site.baseurl}}assets/images/blogs/2014-03/300d88af-838d-46e6-b4e1-a5f44f57c715.png 484w, http://offering.solutions/wp-content/uploads/2014/03/5-300x165.png 300w" sizes="(max-width: 484px) 100vw, 484px" /></a>

The last piece which fits everything together to get a real feeling of client-based-architecture is the routing, which you can navigate to your application with without having the feeling there is a server behind.

<a href="{{site.baseurl}}assets/images/blogs/2014-03/874e9c02-d238-4cc9-afca-686785cd9397.png" rel="attachment wp-att-1151"><img class="aligncenter size-full wp-image-1151" src="{{site.baseurl}}assets/images/blogs/2014-03/874e9c02-d238-4cc9-afca-686785cd9397.png" alt="6" width="938" height="141" srcset="{{site.baseurl}}assets/images/blogs/2014-03/874e9c02-d238-4cc9-afca-686785cd9397.png 938w, http://offering.solutions/wp-content/uploads/2014/03/6-300x45.png 300w, http://offering.solutions/wp-content/uploads/2014/03/6-768x115.png 768w" sizes="(max-width: 938px) 100vw, 938px" /></a>

With this angular.js-concept and WebAPI in the back you can get your information online, take it to your client and work with it. Navigation is client-based. Every behavior but the source of information is based on the client.

&nbsp;

**Conclusion**

Let’s summarize: The patterns to build up applications with a good architecture have been there so far. Since many years. Nowadays this grows further and further into JavaScript and client-side-applications. I think the word &#8220;Application&#8221; has to be paid the most attention to. Frameworks like angular.js are very into giving you the well knows structures and architecture-patterns to build up applications with a lightweight architecture completely using the client.

On the mentioned historically view every C#-Developer has to face the new age with JavaScript as a language you have to pay attention to.

Personally I think now that JavaScript is not a language without structure etc. It’s not like your good old C# with namespaces and classes and stuff but the frameworks in this direction are growing and getting better and better. So here is something moving. There is something growing. Let’s grow with it!

&nbsp;
