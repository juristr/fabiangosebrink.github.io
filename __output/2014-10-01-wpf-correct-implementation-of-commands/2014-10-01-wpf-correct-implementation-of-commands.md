---
id: 475
title: 'WPF &#8211; Correct implementation of Commands'
date: 2014-10-01T17:32:42+00:00
author: Fabian Gosebrink
layout: post
tags: architecture commands wpf 
logo: 'assets/images/logo_small.png'
navigation: True
cover: 'assets/images/download_edit_dark.jpg'
subclass: 'post tag-speeches'
---

# WPF Correct implementation of Commands

Again we will take the code from the posts before ([Wpf Basics I](http://offering.solutions/2014/09/02/wpf-basics-how-to-make-first-steps-of-databinding/ "[Update] Wpf Basics I – How to make first steps of Databinding") and [Wpf Basics II](http://offering.solutions/2014/09/14/wpf-basics-ii-inotifypropertychanged/ "Wpf Basics II – INotifyPropertyChanged")) and make it nicer. Well: What we did was databind a string to the UI. Then we implemented a solution for notify the UI that something changed. Everything triggered with a small timer which was doing the work for &#8220;I have something new&#8221;. But what if the user himself wants to trigger the event for &#8220;I want something to be changed&#8221;?

Therefore we have commands. Commands can maybe triggered from Buttons and are doing some work. In our case we will now remove the timer and give our UI a button to trigger the changes.

So first we get rid of our timer in the NameProvider:

<pre class="lang:c# decode:true">public class NameProviderImpl : NotifyPropertyChangedBase, INameProvider
{
	private string _nameToDisplay;

	public string NameToDisplay
	{
		get
		{
			return _nameToDisplay;
		}
		set
		{
			_nameToDisplay = value;
			OnPropertyChanged(() =&gt; NameToDisplay);
		}
	}
}</pre>

Then we will do a namespace for our commands and add one

![alttext]({{site.baseurl}}assets/images/blogs/2014-09/4c82562d-11aa-435e-acac-92d706201c7a.jpg)Important for the command is: Let it be a single class. It makes testing easier and decouples it from the viewmodel. Also let it inherit from ICommand and implement this interface. And let it inherit from your command Interface 😉

<pre class="lang:c# decode:true ">public class MyCommandImpl : ICommand, IMyCommand
{
	private readonly INameProvider _nameProvider;

	public bool CanExecute(object parameter)
	{
		return true;
	}

	public void Execute(object parameter)
	{
		_nameProvider.NameToDisplay = "Hallelujah";
	}

	public MyCommandImpl(INameProvider nameProvider)
	{
		_nameProvider = nameProvider;
	}

	public event EventHandler CanExecuteChanged;
}</pre>

&nbsp;

<pre class="lang:c# decode:true">public interface IMyCommand
{
	bool CanExecute(object parameter);
	void Execute(object parameter);
	event EventHandler CanExecuteChanged;
}</pre>

So what the command offers us are two methods: &#8220;CanExecute&#8221; is giving us a bool indicating if the command can be executed or not. Based on this the button will be greyed out or not. The &#8220;CanExecuteChanged&#8221; can be fired, if something, which the CanExecute is based on, has changed and it has to be evaluated again. The button may appear not greyed out in the UI after firing this. And the execute is for executing the Command. What a surprise 😉

The command is only taking the nameprovider and setting the name. Exactly what the timer did before.

> The name &#8220;Name**Provider**&#8221; does not match here. Because it does not provide a name to us. It is more a service. Image you would trigger like a calculation or something 😉

So now let the viewmodel offer the command that it can be triggered from the outside:

<pre class="lang:c# decode:true ">public class MainViewModel
{
	public INameProvider NameProvider { get; set; }
	public IMyCommand MyCommand { get; set; }

	public MainViewModel()
	{
		NameProvider = new NameProviderImpl();
		MyCommand = new MyCommandImpl(NameProvider);
	}
}</pre>

Now we offer the UI a NameProvider to bind on and a Command to bind on. The only thing we now have to do is bind this button to the command:

<pre class="lang:c# decode:true ">&lt;Grid&gt;
	&lt;StackPanel&gt;
		&lt;TextBlock Text="{Binding NameProvider.NameToDisplay}"&gt;&lt;/TextBlock&gt;
		&lt;Button Command="{Binding MyCommand}" Height="20"&gt;&lt;/Button&gt;
	&lt;/StackPanel&gt;
&lt;/Grid&gt;</pre>

Now the command is getting executed from the button. It sets the name in the nameprovider and fires the event, that something has changed and the UI gets updated.

That should be it for the very basic of WPF.

&nbsp;

Solution can be loaded here: [DataBindingGettingStarted](http://offering.solutions/wp-content/uploads/2014/09/DataBindingGettingStarted.zip)

Enjoy

Fabian
