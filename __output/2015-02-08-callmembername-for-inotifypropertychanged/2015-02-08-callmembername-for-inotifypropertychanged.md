---
id: 589
title: WPF CallMemberName for INotifyPropertyChanged
date: 2015-02-08T23:08:26+00:00
author: Fabian Gosebrink
layout: post
tags: callmembername inotifypropertychanged wpf 
logo: 'assets/images/logo_small.png'
navigation: True
cover: 'assets/images/download_edit_dark.jpg'
subclass: 'post tag-speeches'
---

# WPF CallMemberName for INotifyPropertyChanged

Regarding to this blogpost [# WPF CallMemberName for INotifyPropertyChanged

Regarding to this blogpost](http://offering.solutions/2014/09/14/wpf-basics-ii-inotifypropertychanged/ "Wpf Basics II – INotifyPropertyChanged") I showed up how to implement the INotifyPropertyChanged.

Well with .Net-Framework 4.5 a new solution came up to make it even more simple:

Instead of the &#8220;old&#8221; (but working) base-class

<pre class="lang:c# decode:true ">public class NotifyPropertyChangedBase : INotifyPropertyChanged
    {
        public event PropertyChangedEventHandler PropertyChanged;

        protected virtual void OnPropertyChanged&lt;T&gt;(Expression&lt;Func&lt;T&gt;&gt; propertyExpression)
        {
            MemberExpression memberExpr = propertyExpression.Body as MemberExpression;
            if (memberExpr == null)
            {
                throw new ArgumentException("propertyExpression should represent access to a member");
            }

            PropertyChangedEventHandler handler = PropertyChanged;
            if (handler != null)
            {
                handler(this, new PropertyChangedEventArgs(memberExpr.Member.Name));
            }
        }
    }</pre>

The OnPropertyChaned can be replaced with:

<pre class="lang:c# decode:true ">public class NotifyPropertyChangedBase : INotifyPropertyChanged
    {
        public event PropertyChangedEventHandler PropertyChanged;

        protected void NotifyPropertyChanged([CallerMemberName] string propertyName = null)
        {
            PropertyChangedEventHandler propChanged = PropertyChanged;
            if (propChanged != null)
            {
                propChanged(this, new PropertyChangedEventArgs(propertyName));
            }
        }
    }</pre>

The CallMemberName-Attribute is automatically set to the PropertyName which calls it. So the usage in the ViewModel (or whereever you throw the event) is now a simple:

<pre class="lang:c# decode:true ">public string NameToDisplay
        {
            get
            {
                return _nameToDisplay;
            }
            set
            {
                _nameToDisplay = value;
                NotifyPropertyChanged();
            }
        }</pre>

See: You do not need to give a func. instead you are only calling the method whithout any paramters.

Solution with a command-implementation can be found here: [DataBindingGettingStarted](http://offering.solutions/wp-content/uploads/2015/02/DataBindingGettingStarted.zip)

Happy coding!

&nbsp;

Regards