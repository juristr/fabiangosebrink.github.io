---
id: 463
title: 'Wpf - The INotifyPropertyChanged Interface'
date: 2014-09-14T19:41:48+00:00
author: Fabian Gosebrink
layout: post
tags: basics notifypropertychanged wpf 
logo: 'assets/images/logo_small.png'
navigation: True
cover: 'assets/images/download_edit_dark.jpg'
subclass: 'post tag-speeches'
disqus: true
categories: articles
---

UPDATE UPDATE UPDATE:

Check the CallMemberName-Possibility [here](http://offering.solutions/2015/02/08/callmembername-for-inotifypropertychanged/ "CallMemberName for INotifyPropertyChanged")

In the first part I told something about the databinding [here](http://offering.solutions/2014/09/02/wpf-basics-how-to-make-first-steps-of-databinding/ "Wpf Basics I – How to make first steps of Databinding"). The second part should be something about refreshing the data at the UI. We said that the UI only knows the datacontext and its properties. So far so god. It is binding them at startup and we&#8217;re done so far.

Just to calm down the ones who expect a solution: Can be found in the third article [here](http://offering.solutions/2014/10/01/wpf-basics-iii-correct-implementation-of-commands/ "Wpf Basics III – Correct implementation of Commands")

But what if the data underneath is changing. What if a service or anything else has new data and want to tell the UI "Hey there, I have something new!"

Therefore the binding has to be "refreshed" and we have the INotifyPropertyChanged-Interface to get this job done.

Lets take our code from before and give it a timer which sets the name we want to display after 3 seconds:

<pre class="lang:c# decode:true ">&lt;Grid&gt;
	&lt;StackPanel&gt;
		&lt;TextBlock Text="{Binding NameToDisplay}"&gt;&lt;/TextBlock&gt;
	&lt;/StackPanel&gt;
&lt;/Grid&gt;</pre>

&nbsp;

<pre class="lang:c# decode:true ">public class MainViewModel
{
	public string NameToDisplay { get; set; }
	Timer _timer;

	public MainViewModel()
	{
		_timer = new Timer(3000);
		_timer.Elapsed += timer_Elapsed;
		_timer.Enabled = true;
	}

	void timer_Elapsed(object sender, ElapsedEventArgs e)
	{
		NameToDisplay = "Hallelujah";
		_timer.Enabled = false;
	}
}</pre>

>  &#8220;Hallelujah&#8221; is always my testword because im pretty sure it occurs nowhere else in a solution 😉 So if you see this, its mine

So, if you debug this you will see that the timer gets into the timer_elapsed-function and sets the name but the UI does not change. So lets implement a way to refresh the UI! Only implement the INotifyPropertyChanged-interface:

<pre class="lang:c# decode:true">public class MainViewModel : INotifyPropertyChanged
{
	public string NameToDisplay { get; set; }
	readonly Timer _timer;

	public MainViewModel()
	{
		_timer = new Timer(3000);
		_timer.Elapsed += timer_Elapsed;
		_timer.Enabled = true;
	}

	void timer_Elapsed(object sender, ElapsedEventArgs e)
	{
		NameToDisplay = "Hallelujah";
		_timer.Enabled = false;

		if (PropertyChanged != null)
		{
			PropertyChanged(this, new PropertyChangedEventArgs("NameToDisplay"));
		}
		
	}

	public event PropertyChangedEventHandler PropertyChanged;
}</pre>

So everthing we do is throwing the event that something has changed with the name of the property as a string. If you let this run you will see that the UI refreshes and after 3 seconds the &#8220;hallelujah&#8221; is displayed. But this has some disadvantages:

  * We are throwing the event in the timer_elapsed. So only when **this** is done the property is refreshed
  * We are having the name of the property as a string in it. So renaming the property will mostly NOT rename the string. (Magic String). And the refresh does not work again.
  * Refreshing the UI is a base function. It should be outsourced in like a base file or something.

Lets tune this:

  1. First we will make a namespace for this (I love namespaces) called &#8220;Common&#8221; and make a basefile in there.
  2. We will make this function generic expecting a lambda-Expression to erase the magic string
  3. We will call the refreshing thing in the setter of the property itself. Then its getting refreshed everytime someone in the code sets it.

&nbsp;

![alttext]({{site.baseurl}}assets/articles/2014-09-14/0797f3e5-c8c3-4ed4-ab00-889e2bd8c628.jpg)which looks like:

<pre class="lang:c# decode:true">public class NotifyPropertyChangedBase : INotifyPropertyChanged
{
	public event PropertyChangedEventHandler PropertyChanged;

	protected virtual void OnPropertyChanged&lt;T&gt;(Expression&lt;Func&lt;T&gt;&gt; propertyExpression)
	{
		MemberExpression memberExpr = propertyExpression.Body as MemberExpression;
		if (memberExpr == null)
		{
			throw new ArgumentException("Wrong PropertyExpression");
		}

		PropertyChangedEventHandler handler = PropertyChanged;
		if (handler != null)
		{
			handler(this, new PropertyChangedEventArgs(memberExpr.Member.Name));
		}
	}
}</pre>

This is taking the member and throwing the event for us on this member. That was Point 1 and 2. Let it be (three)!

We do inherit from the just created class and can access the event with the lambda-expression, which is more generic:

<pre class="lang:c# decode:true">public class MainViewModel : NotifyPropertyChangedBase
{
	readonly Timer _timer;
	private string _nameToDisplay;

	public string NameToDisplay
	{
		get { return _nameToDisplay; }
		set
		{
			_nameToDisplay = value;
			OnPropertyChanged(() =&gt; NameToDisplay);
		}
	}

	public MainViewModel()
	{
		_timer = new Timer(3000);
		_timer.Elapsed += timer_Elapsed;
		_timer.Enabled = true;
	}

	void timer_Elapsed(object sender, ElapsedEventArgs e)
	{
		NameToDisplay = "Hallelujah";
		_timer.Enabled = false;
	}
}</pre>

Remeber: In the elapsed-method we are setting the property (not the private variable) directly that the setter is called and the event is thrown.

> This is not the only way to implement this. This can be done in several ways. But for beginners this should do the trick.

Run it, it will show you the text after three seconds.

If you want to, read further how you can get this cleaner with services and erase the NotifyPropertyChangedBase from the viewmodel.

Lets tune this a little bit: The viewmodel does a lot of work. It does not have to do this, so lets extract this a bit and make it more clean.

First we do a NameProvider, which gives us the name. In my case again with a timer to see the UI changing. Normally this could be a service or something else without a timer. Could be anything which triggers the UI to change (not only) after a piece of work.

![alttext]({{site.baseurl}}assets/articles/2014-09-14/55c55993-75bd-4e1b-924b-50ae54555462.jpg)

<pre class="lang:c# decode:true ">public class NameProviderImpl : NotifyPropertyChangedBase, INameProvider
    {
        private readonly Timer _timer;
        private string _nameToDisplay;

        public string NameToDisplay
        {
            get
            {
                return _nameToDisplay;
            }
            private set
            {
                if (_nameToDisplay == value)
                {
                    return;
                }

                _nameToDisplay = value;
                OnPropertyChanged(() =&gt; NameToDisplay);
            }
        }

        public NameProviderImpl()
        {
            _timer = new Timer(3000);
            _timer.Elapsed += timer_Elapsed;
            _timer.Enabled = true;
        }

        void timer_Elapsed(object sender, ElapsedEventArgs e)
        {
            NameToDisplay = "Hallelujah";
            _timer.Enabled = false;
        }
    }</pre>

<pre class="lang:c# decode:true ">public interface INameProvider
{
	string NameToDisplay { get; }
}</pre>

Everything we did here is moving the timer-logic into a provider and offering the property through an interface to the outside.

Our viewmodel now has nearly no logic anymore:

<pre class="lang:c# decode:true ">public class MainViewModel
{
	public INameProvider NameProvider { get; set; }

	public MainViewModel()
	{
		NameProvider = new NameProviderImpl();
	}
}</pre>

This principle I am also describing [here](http://offering.solutions/2014/07/03/wpf-introducing-services-in-the-viewmodel-viewmodel-as-facade/ "WPF – Introducing services in the viewmodel (viewmodel as facade)").

Now we have to change the binding a bit. Because now the viewmodel is giving us the property to bind not directly but onto another property &#8220;NameProvider&#8221;. So the Binding looks like this:

<pre class="lang:c# decode:true">&lt;Grid&gt;
	&lt;StackPanel&gt;
		&lt;TextBlock Text="{Binding NameProvider.NameToDisplay}"&gt;&lt;/TextBlock&gt;
	&lt;/StackPanel&gt;
&lt;/Grid&gt;</pre>

Run this and you will see the result stays the same: After three seconds our string is displayed.

![alttext]({{site.baseurl}}assets/articles/2014-09-14/3fa5c15f-bde9-4a8c-a970-6384d98850e0.jpg)So what we did now is: Getting our Viewmodel nice and clean. It gives us an overview of services and providers which the UI can use. It does not inherit from NotifyPropertyChangedBase. You saw how flexible databinding is. Not only with strings but you can bind also lists of objects etc.

&nbsp;
