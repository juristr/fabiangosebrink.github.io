---
id: 1264
title: Writing custom validators in Angular2
date: 2016-05-10T17:42:23+00:00
author: Fabian Gosebrink
layout: post
tags: angular2 custom validator 
logo: 'assets/images/logo_small.png'
navigation: True
cover: 'assets/images/download_edit_dark.jpg'
subclass: 'post tag-speeches'
disqus: true
categories: articles
---

In this blogpost I want to show you how you are writing custom validators in Angular 2.

<a href="https://github.com/FabianGosebrink/ASPNET-ASPNET-Core-Angular1-Angular2-Demo/tree/master/Angular2-Client">https://github.com/FabianGosebrink/ASPNET-ASPNET-Core-Angular1-Angular2-Demo/tree/master/Angular2-Client</a>

<blockquote>Updated to Angular 2.0 Syntax</blockquote>

When dealing with forms in Angular2 you can use the build-in validators like: required, minLength or maxLength, pattern... . Sooner or later you need a custom validator fulfilling a special validation.

Let's have a form first:

<pre class="lang:c# decode:true">&lt;form #f="ngForm"&gt;

    &lt;div class="form-group"&gt;
        &lt;label for="calories"&gt;Calories&lt;/label&gt;
        &lt;input type="text" class="form-control" id="calories" placeholder="Calories" [(ngModel)]="foodItem.Calories"
             name="calories"&gt;
   
    &lt;/div&gt;
    &lt;button type="button" class="btn btn-default" (click)="AddOrUpdateFood()" [disabled]="!f?.valid"&gt;Submit&lt;/button&gt;
&lt;/form&gt;</pre>

We see a form which has a variable "f" representing the form with which we can ask for the current state. We also have an input control in which we can add a number - calories in this case.

<blockquote>I know that we could change the type of the input to number but in this post we want to check if whether the things the user typed are number or not.</blockquote>

So lets write a validator which exactly checks for that:

<pre class="lang:js decode:true">import { Directive, forwardRef, Attribute } from '@angular/core';
import { Validator, FormControl, NG_VALIDATORS } from '@angular/forms';

@Directive({
    selector: '[isNumber][formControlName],[isNumber][formControl],[isNumber][ngModel]',
    providers: [
        { provide: NG_VALIDATORS, useExisting: forwardRef(() =&gt; IsNumberValidator), multi: true }
    ]
})

export class IsNumberValidator implements Validator {

    validate(c: FormControl): { [key: string]: any } {

        if (isNaN(+c.value)) {
            // console.log(c.value + " is not a number");
            return {
                isNumber: false
            }
        }

        // console.log(c.value + " is a number");
        return null;
    }
}</pre>

This validator is named "IsNumberValidator" and has a function with a FormControl as a parameter. After the imports we define a directive here and adding the selectors we want. The "providers" let us extend the build-in NG_VALIDATORS and add our new validator. The class implements the validator class with the "validate"-Method which takes a FormControl and returns null if everything is okay and not null if the validation fails.

&nbsp;

Before we can use it in our form we have to include it in our module. I built a shared module which I include in my app.module.

<pre class="lang:js decode:true ">import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';

import { IsNumberValidator } from '../validators/isNumber.validator';
import { IsInRangeValidator } from '../validators/isInRange.validator';

@NgModule({
    imports: [
        // Modules
        BrowserModule
    ],

    declarations: [

        // Components &amp; directives

        IsNumberValidator,
         IsInRangeValidator
    ],

    providers: [
        // Services
    ],

    exports: [
        IsNumberValidator,
        IsInRangeValidator
    ]
})

export class SharedModule { }</pre>

and in our app.module:

<pre class="lang:js decode:true ">// ...

import { SharedModule } from './modules/shared.module';


@NgModule({
    imports: [
        // ...
        SharedModule
    ],

    declarations: [
        // ...
    ],

    providers: [
        // ...
    ],

    bootstrap: [AppComponent]
})

export class AppModule { }</pre>

&nbsp;

We can use it in the form like

<pre class="lang:c# decode:true">&lt;form #f="ngForm"&gt;
    &lt;div class="form-group"&gt;
        &lt;label for="calories"&gt;Calories&lt;/label&gt;
        &lt;input type="text" class="form-control" id="calories" placeholder="Calories" [(ngModel)]="foodItem.Calories"
            isNumber name="calories"&gt;
   
    &lt;/div&gt;
    &lt;button type="button" class="btn btn-default" (click)="AddOrUpdateFood()" [disabled]="!f?.valid"&gt;Submit&lt;/button&gt;
&lt;/form&gt;</pre>

&nbsp;

Here we add the validator to the input field of our form. It is only a directive so we can treat it this way like we did with every directive before.

So now the validator directive fires everytime a key is hit. We can now add a

<pre class="lang:js decode:true">... #calories="ngModel" ...</pre>

to the form and check if the variable we introduced is valid with

<pre class="lang:c# decode:true ">... calories.isValid ...</pre>

<pre class="lang:xhtml decode:true ">&lt;form #f="ngForm"&gt;
    &lt;div class="form-group"&gt;
        &lt;label for="calories"&gt;Calories&lt;/label&gt;
        &lt;input type="text" class="form-control" id="calories" placeholder="Calories" [(ngModel)]="foodItem.Calories"
            isNumber name="calories" #calories="ngModel"&gt;
   
    &lt;div *ngIf="!calories.valid" class="alert alert-danger"&gt;
            Field is not valid
        &lt;/div&gt;
   
    &lt;/div&gt;
    &lt;button type="button" class="btn btn-default" (click)="AddOrUpdateFood()" [disabled]="!f?.valid"&gt;Submit&lt;/button&gt;
&lt;/form&gt;</pre>

&nbsp;

But it would be nice to show <em>specific</em> error messages to the users and not only if the filed is valid or not. So we have to ask through the form-variable we introduced and through the control name for the state and if there is an error. We can do this through the "hasError" method. Combining it with the dirty-check and our validator-naming we can show the user messages exactly to what was going wrong in the form.

&nbsp;

<pre class="lang:c# decode:true ">&lt;form #f="ngForm"&gt;

    &lt;div class="form-group"&gt;
        &lt;label for="calories"&gt;Calories&lt;/label&gt;
        &lt;input type="text" class="form-control" id="calories" placeholder="Calories" [(ngModel)]="foodItem.Calories"
            required isNumber name="calories"&gt;
        
        &lt;div *ngIf="f?.controls?.calories?.hasError('required') &amp;&amp; f?.controls?.calories?.dirty" class="alert alert-danger"&gt;
            Pflichtfeld!
        &lt;/div&gt;

        &lt;div *ngIf="f?.controls?.calories?.hasError('isNumber') &amp;&amp; f?.controls?.calories?.dirty" class="alert alert-danger"&gt;
            Field is not a number
        &lt;/div&gt;
    &lt;/div&gt;
    &lt;button type="button" class="btn btn-default" (click)="AddOrUpdateFood()" [disabled]="!f?.valid"&gt;Submit&lt;/button&gt;
&lt;/form&gt;

</pre>

And that's basically it.

![Angular2CustomValidators]({{site.baseurl}}assets/articles/2016-05-10/5eac1264-a787-43be-9144-58fc0fec9ef4.jpg)

HTH

Regards

<https://github.com/FabianGosebrink/Angular1-And-Angular2-Tutorials-With-Demo-App/tree/master/Angular2-Tutorial/05_Forms/end>

<http://blog.thoughtram.io/angular/2016/03/14/custom-validators-in-angular-2.html>

Thanks to [Jürgen Gutsch](http://www.gutsch-online.de/) for doing a review of this blogpost.

Fabian


