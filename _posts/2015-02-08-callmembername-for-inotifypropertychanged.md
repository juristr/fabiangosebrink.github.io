---
id: 589
title: WPF CallMemberName for INotify Property Changed
date: 2015-02-08T23:08:26+00:00
author: Fabian Gosebrink
layout: post
tags: callmembername inotifypropertychanged wpf 
logo: 'assets/images/logo_small.png'
navigation: True
cover: 'assets/images/download_edit_dark.jpg'
subclass: 'post tag-speeches'
disqus: true
categories: articles
---

Regarding to this blogpost [# WPF CallMemberName for INotify Property Changed](http://offering.solutions/2014/09/14/wpf-basics-ii-inotifypropertychanged/) I showed up how to implement the INotifyPropertyChanged.

Well with .Net-Framework 4.5 a new solution came up to make it even more simple:

Instead of the "old" (but working) base-class

```
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
```

The OnPropertyChaned can be replaced with:

```
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
```

The CallMemberName-Attribute is automatically set to the PropertyName which calls it. So the usage in the ViewModel (or whereever you throw the event) is now a simple:

```
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
```

See: You do not need to give a func. instead you are only calling the method whithout any paramters.

Solution with a command-implementation can be found here: [DataBindingGettingStarted](http://offering.solutions/wp-content/uploads/2015/02/DataBindingGettingStarted.zip)

Happy coding!

Regards
