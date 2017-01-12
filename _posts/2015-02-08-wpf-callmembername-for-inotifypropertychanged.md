---
id: 986
title: WPF CallMemberName for INotifyPropertyChanged
date: 2015-02-08 23:08
author: Fabian Gosebrink
layout: post
tags: CallMemberName INotifyPropertyChanged WPF
logo: 'assets/images/logo_small.png'
navigation: True
cover: 'assets/images/aerial-view-of-laptop-and-notebook_bw_osc.jpg'
subclass: 'post tag-speeches'
disqus: true
categories: blog articles
---

In this blogpost I want to show you how to use WPF CallMemberName for INotifyPropertyChanged.

Regarding to this blogpost [here](http://offering.solutions/2014/09/14/WPF-basics-ii-inotifypropertychanged/) I showed up how to implement the INotifyPropertyChanged.

Well with .Net-Framework 4.5 a new solution came up to make it even more simple:

Instead of the "old" (but working) base-class

{% highlight cs %}
public class NotifyPropertyChangedBase : INotifyPropertyChanged
{
    public event PropertyChangedEventHandler PropertyChanged;

    protected virtual void OnPropertyChanged<T>(Expression<Func<T>> propertyExpression)
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
}
{% endhighlight %}

The OnPropertyChanged can be replaced with:

{% highlight cs %}
public class NotifyPropertyChangedBase : INotifyPropertyChanged
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
}
{% endhighlight %}

The CallMemberName-Attribute is automatically set to the PropertyName which calls it. So the usage in the ViewModel (or whereever you throw the event) is now a simple:

{% highlight cs %}
public string NameToDisplay
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
}
{% endhighlight %}

See: You do not need to give a func. instead you are only calling the method whithout any paramters.

Solution with a command-implementation can be found here: [DataBindingGettingStarted]({{site.baseurl}}assets/articles/wp-content/uploads/2015/02/DataBindingGettingStarted.zip)

Happy coding!

Regards
