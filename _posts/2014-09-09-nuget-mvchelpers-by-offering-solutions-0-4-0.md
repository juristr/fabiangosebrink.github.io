---
id: 513
title: '[NuGet] &#8211;  MvcHelpers by Offering.Solutions 0.4.0'
date: 2014-09-09T17:55:08+00:00
author: Fabian Gosebrink
layout: post
tags: asp.netmvc helpers 
logo: 'assets/images/logo_small.png'
navigation: True
cover: 'assets/images/download_edit_dark.jpg'
subclass: 'post tag-speeches'
disqus: true
categories: articles
---

Hey,

I just created the third NuGet

[Nuget: MvcHelpers by Offering.Solutions 0.4.0](https://www.nuget.org/packages/OfferingSolutions.MvcHelpers/0.4.0">Nuget: MvcHelpers by Offering.Solutions 0.4.0)

which gives you really really small helper functions (not only) for your mvc projects.

Features are:
  
  * Nullcheck with Exception
  * Nullcheck with returning the type if not null, otherwise exception
  * Return bool value if something is null
  * Checking enumerable if its null or empty
  * Checks if a string is a timespan
  * Checks if a string is an email
  * Converts a string to datetime
  * Converts s tring to timespan
  * Checks a string for null, empty or a given lenght (e.g. for password purposes)
  * Checks if a HttpPostedFile has a specific format
  * Checks if a HttpPostedFile is an image
  * Checks if a string can be converted to double
  * Converts a string to double

    I think these little helpers can save you a lot of short but necessary work in your projects.

  <pre class="lang:c# decode:true ">static void Main(string[] args)
{
	//Throws an exception
	Person person = null;
	person.CheckIsNull();

	// Throws an exception if null, gives you back the type otherwise --&gt; Good for DI in public methods
	Person myPerson = person.CheckIsNull("Person must not be null");

	// Is true in this case
	bool personIsNull = person.IsNull();

	person = new Person() {Id = 1, Name = "Fabian"};
	List&lt;Person&gt; listOfPersons = new List&lt;Person&gt;() {person};

	//should be false
	bool isNullOrEmpty = listOfPersons.IsNullOrEmpty();

	// Checks a string for a valid TimeSpan
	string timeSpanToCheck = "11:20";
	bool isTimeSpan = timeSpanToCheck.IsTimeSpan();

	// Checks if a string is a valid email or not
	string email = "dont@ask.me";
	bool isEmail = email.IsEmail();

	//Converts a string to datetime
	string toDateTime = "01.01.2010";
	DateTime dateTime = toDateTime.ToDateTime();

	// COnverts a string to Timespan
	timeSpanToCheck = "11:20";
	TimeSpan timeSpan = timeSpanToCheck.ToTimeSpan();

	//Cheks, if a password has a specific length or is null or empty
	string password = "TheseAreAtLeastEightSigns";
	bool isValidPassword = password.IsValidPassword(8);

	// Checks if a file has a given format...
	HttpPostedFileBase httpPostedFile = null;//...
	bool hasFormat = httpPostedFile.HasFormat("txt");

	// ... or wth multiple extensions
	hasFormat = httpPostedFile.HasFormat("txt", "pdf");

	// Checks if a file is an image...
	bool isImage = httpPostedFile.IsImage();

	// ...also with formats
	isImage = httpPostedFile.IsImage("jpg", "gif");

	//Cheks if a value is a double value (separation with . or ,
	string doubleToCheck = "3.5";
	bool isDouble = doubleToCheck.IsDouble();

	//COnverts a string to double
	doubleToCheck = "3.5";
	double doubleValue = doubleToCheck.ToDouble();
}</pre>
  
If you encounter any mistakes, please feel free to leave a comment.

Enjoy

Fabian
