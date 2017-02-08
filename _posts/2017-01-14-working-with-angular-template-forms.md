---
title: Working with Angular Template Forms
date: 2017-01-14 13:49
author: Fabian Gosebrink
layout: post
tags: angular forms templatedriven
logo: 'assets/images/logo_small.png'
navigation: True
cover: 'assets/images/aerial-view-of-laptop-and-notebook_bw_osc.jpg'
subclass: 'post tag-speeches'
disqus: true
categories: blog articles
---

In this blog I want explain and show you how to work with Angular's template driven forms.

### The Problem

I was recently giving an Angular course to awesome people and I noticed that working with Angular forms seems to be one of the most complicated things in Angular.

### The Situation

We will start with a simple Angular application which has a structure like this:

```
app
├── app.component.ts
├── app.component.html
├── form.component.ts
├── form.component.html
├── app.module.ts
└── main.ts
index.html
systemjs.config.js
tsconfig.json
tslint.json
styles.css
package.json
```

this is an easy way to start with Angular and template forms. 

> I did not mention files like favicon etc. They do not really impact our work here

Now lets got create a simple form. For this I will grab bootstraps most simple form just to get a nice look and feel. 

[Bootstraps Forms](http://getbootstrap.com/css/#forms)

Now we add the bootstrap cdn to our ```index.html```.

{% highlight html %}
{% raw %}

<head>
  <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css">
</head>

{% endraw %}
{% endhighlight %}

Out app.component.html simply looks like this:

{% highlight html %}
{% raw %}

<div class="container">
    <h1>{{title}}</h1>
    <form-component></form-component>
</div>

{% endraw %}
{% endhighlight %}

### Introducing the Form

We remove the checkbox and the file upload from the copied form from bootstrap just for clarity and add the form template to our form component. Further we want to handle a simple Name for understanding. Lets modify the template to only work with a name.

So our ```form.component.html``` looks like this now:

{% highlight html %}

<form>
    <div class="form-group">
        <label for="exampleInputName">Name</label>
        <input type="name" class="form-control" id="exampleInputName" placeholder="Name">
    </div>
    <button type="submit" class="btn btn-default">Submit</button>
</form>

{% endhighlight %}

So what we did here so far is nothing Angular related! We just added simple HTML-Form things.

We will now go ahead and make this thing an Angular form.

### Add Angular to the form

#### The model

We will use a normal customer to deal with the form. Also nothing spectecular here, we just add a class representing this customer:

{% highlight ts %}

export class Customer {
    constructor(public name: string) { }
}

{% endhighlight %}

and we place it in a special folder called ```models```

```
app
├── models
    └──  customer.model.ts
├── app.component.ts
├── app.component.html
├── form.component.ts
├── form.component.html
├── app.module.ts
└── main.ts
index.html
systemjs.config.js
tsconfig.json
tslint.json
styles.css
package.json
```

Then we can make a new model in the components constructor like this

{% highlight ts %}

export class FormComponent {
  public title: string = 'Template Forms';
  public model: Customer;

  constructor() {
    this.model = new Customer('Fabian');
  }
}

{% endhighlight %}

#### Adding the binding to the model with ngModel

Next thing is to add the binding to the inputs in our forms. And here is the first time we marry Angular with our html.

We take the ```ngModel```-directive to tell the field "Hey, this thing I give you here is important for you form. Please, my little form, take care of it"

{% highlight html %}

<div class="container">
    <!--...-->
     <input type="name" class="form-control" id="exampleInputName" placeholder="Name" ngModel>
    <!--...-->
</div>

{% endhighlight %}

Now this input is marked with the ```ngModel```-directive. But what is that good for?

Before we go on clarify that can also take care of something Angular prepared for us. What we want to do now is asking our form about its state. What is the state of our form currently and when the user types something in? We can not access our form yet, because we do not have a name to ask for. So let's introduce one!

We can do this by adding a variable in the template with the "#"-Sign and tagging the form with the ```ngForm```-directive Angular prepared for us.

This means:

{% highlight html %}

<form>

{% endhighlight %}

becomes to

{% highlight html %}

<form #myform="ngForm">

{% endhighlight %}

We have introduced a variable and assign the complete form to it. Angular introduced this ```ngForm``` for us in the background, we are only catching it up and take a variable to make it accessible. Now we are able to ask the form for its state! This variable is called a template reference variable.

What we want is to see the state of our form. We achieve that by adding the ngModel-directive, by giving the form a variable we can access it and one last piece is missing. If we use ngModel in an form control, we also have to tell the Angular form how it should be accessed. So we need a name. Lets introduce a name that this control can be registered onto the form under that name.

{% highlight html %}

<div class="container">
    <!--...-->
     <input type="text" class="form-control" id="exampleInputName" placeholder="Name" ngModel name="name">
    <!--...-->
</div>

{% endhighlight %}

With those three things we can ask the form about its state with

` myform.value `

like this:

{% highlight html %}
{% raw %}

<p> {{ myform.value | json }} </p>

<form #myform="ngForm">
    <div class="form-group">
        <label for="exampleInputName">Name</label>
        <input type="text" class="form-control" id="exampleInputName" placeholder="Name" ngModel name="name">
    </div>
    <button type="submit" class="btn btn-default">Submit</button>
</form>

{% endraw %}
{% endhighlight %}

The ngModel directive is forcing Angular to persist the state of the form to the form object. So we read it with 

{% highlight js %}
{% raw %}
{{ myform.value | json }}
{% endraw %}
{% endhighlight %}

But further to that it is doing nothing. We got our state only persisted to the form. We want to interact with our model, right? 

Therefore we have to intruduce some kind of databinding. As we know that databinding is working with () and/or [] we have no databinding going on here so far.

We could do the property binding like this

{% highlight html %}

 <input ... [ngModel]="model.name" name="name">

{% endhighlight %}

Then Angular would take this ngModel directive and connect it to a property on our component called "model.name". 

> Remember we created a model on our component with its constructor

When the initial value is set from the component the value would be reflected into the input field. But this is only one way binding! When the value in the input changes it would not be reflected into the model. The form would change its state but the model know nothing about it.

To get the model also changed when the user types something in we have to introduce two way binding like this: 

{% highlight html %}

 <input ... [(ngModel)]="model.name" name="name">

{% endhighlight %}

So the forms state would now reflect the changes into the forms state _and_ into the model we want to work with when submitting. In addition to that the initial state is set because we create a model on the constructor.

#### Further steps with the form

We can use the forms template reference variable also to check some properties on the form.

> The template reference variable offers us a form-property which we can check

We can check the following states on the form :

{% highlight js %}
{% raw %}

<!-- if user touched every field of the form -->
<p>{{myform.form.touched}}</p>

<!-- if every field of the form is not touched yet -->
<p>{{myform.form.untouched}}</p>

<!-- field has not been changed value since it is displayed -->
<p>{{myform.form.pristine}}</p>

<!-- user changed the value since it is displayed -->
<p>{{myform.form.dirty}}</p>

<!-- form is valid -->
<p>{{myform.form.valid}}</p>

<!-- form is invalid -->
<p>{{myform.form.invalid}}</p>

{% endraw %}
{% endhighlight %}

Lets face the "valid" and "invalid" for a second. We can apply a normal HTML ```required``` attribute at one control just to have a reason why a form should be valid or invalid.

{% highlight html %}

<form #myform="ngForm">
    <div class="form-group">
        <label for="exampleInputName">Name</label>
        <input type="text" class="form-control" id="exampleInputName" placeholder="Name" [(ngModel)]="model.name" name="name" required>
    </div>
    <button type="submit" class="btn btn-default">Submit</button>
</form>

{% endhighlight %}

So we can go ahead and for example disable the submit button and show a message when the form is not valid.

{% highlight html %}

<form #myform="ngForm">
    <div class="form-group">
        <label for="exampleInputName">Name</label>
        <input type="text" class="form-control" id="exampleInputName" placeholder="Name" [(ngModel)]="model.name" name="name" required>
    </div>

    <span *ngIf="!myform.form.valid">Check your form</span>

    <button type="submit" class="btn btn-default" [disabled]="!myform.form.valid">Submit</button>

</form>

{% endhighlight %}

So this gives us the possibility to ask the form for some variables Angular gives us.

We can also do this whole template variable thing on a control itself and ask the simple control for its state instead of the whole form. The same properties which are valid for the form apply also on the control. Therefore we have to grab our ngModel-directive again and assign it to a template variable to make the control, or better: the model assigned to this control, accessible.

{% highlight html %}

 <input type="text" class="form-control" id="exampleInputName" placeholder="Name" [(ngModel)]="model.name" name="name" required #name="ngModel">

{% endhighlight %}

Now we can ask for the state of this control in the same way like the form:

{% highlight html %}

<form #myform="ngForm">
    <div class="form-group">
        <label for="exampleInputName">Name</label>
        <input type="text" class="form-control" id="exampleInputName" placeholder="Name" [(ngModel)]="model.name" name="name" required #name="ngModel">
    </div>

    <span *ngIf="!name.valid">name is invalid</span>

    <span *ngIf="!myform.form.valid">Check your form</span>

    <button type="submit" class="btn btn-default" [disabled]="!myform.form.valid">Submit</button>

</form>

{% endhighlight %}

#### Submitting the Form

Our button at the end of the form is currently set to submit the form. But we do not catch the submitting so far. So lets do that.

{% highlight html %}
{% raw %}

<h4>Form Value</h4>

<p>{{myform.value | json}}</p>

<form #myform="ngForm" (ngSubmit)="alertMyFormModel()">
    <div class="form-group">
        <label for="exampleInputEmail1">Email address</label>
        <input type="email" class="form-control" id="exampleInputEmail1" placeholder="Email" ngModel name="email">
    </div>
    <div class="form-group">
        <label for="exampleInputPassword1">Password</label>
        <input type="password" class="form-control" id="exampleInputPassword1" placeholder="Password">
    </div>
    <button type="submit" class="btn btn-default">Submit</button>
</form>

{% endraw %}
{% endhighlight %}

We introduced the ngSubmit-directive and assigned it a function we have to implement on our component.

{% highlight ts %}

export class FormComponent {
  public title: string = 'Template Forms';
  public model: Customer;

  constructor() {
    this.model = new Customer('Fabian');
  }

  public alertMyFormModel() {
    alert(JSON.stringify(this.model));
  }
}

{% endhighlight %}

If you now click the button the form gets submitted and you should see the model in a json string in an alert box.

I hope I could clarify some things about Angulars template driven forms.

Best regards

Fabian

