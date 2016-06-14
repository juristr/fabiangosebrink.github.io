---
id: 1264
title: Writing custom validators in Angular2
date: 2016-05-10T17:42:23+00:00
author: Fabian Gosebrink
layout: post
permalink: /writing-custom-validators-angular2/
logo: 'assets/images/logo_small.png'
navigation: True
cover: 'assets/images/download_edit_dark.jpg'
tags:
  - Angular2
  - custom
  - validator
subclass: 'post tag-speeches'
---

Hey,

in this blogpost I want to show you how to write custom validators for forms in Angular2.

> Note: While writing this post the current version of Angular is RC1. So all this examples are written with code using the RC1 version

When dealing with forms in Angular2 you can use the build-in validators like: required, minLength or maxLength, pattern&#8230; . Sooner or later you need a custom validator fulfilling a special validation.

Let&#8217;s have a form first:

<pre class="lang:c# decode:true">&lt;form #myForm="ngForm"&gt;
    &lt;div class="form-group"&gt;
        &lt;label for="inputCalories"&gt;Calories&lt;/label&gt;
        &lt;input type="number" 
            class="form-control" 
            id="inputCalories" 
            ngControl="calories" 
            placeholder="Calories" 
            [(ngModel)]="foodItem.Calories"&gt;
    &lt;/div&gt;
    &lt;button type="button" class="btn btn-default" (click)="AddFood()" [disabled]="!myForm?.form.valid"&gt;Submit&lt;/button&gt;
&lt;/form&gt;</pre>

We see a form which has a variable &#8220;myForm&#8221; with which we can ask for the current state of the form.

Now we want to write a custom validator checking if the entered number is in a specific range for example.

<pre class="lang:c# decode:true ">import { Control } from '@angular/common';

export class IntegerValidator {

    static range(c: Control): any {

        if (c.value &gt; 2147483647 || c.value &lt; 0) {
            return { 'isInRange': false };
        }
        return null;
    }
}</pre>

This validator is named &#8220;IntegerValidator&#8221; and has a function with a control as a parameter. It returns null if everything is okay and not null if the entered number is not in integer range. You can access the control&#8217;s value via [control].value.

If we want to use it in the form like

<pre class="lang:c# decode:true ">&lt;form #myForm="ngForm"&gt;
    &lt;div class="form-group"&gt;
        &lt;label for="inputCalories"&gt;Calories&lt;/label&gt;
        &lt;input type="number" 
            class="form-control" 
            id="inputCalories" 
            ngControl="calories" 
            placeholder="Calories" 
            [(ngModel)]="foodItem.Calories"
            isInRange&gt;
    &lt;/div&gt;
    &lt;button type="button" class="btn btn-default" (click)="AddFood()" [disabled]="!myForm?.form.valid"&gt;Submit&lt;/button&gt;
&lt;/form&gt;</pre>

We need a directive to access the validator this way. Let&#8217;s write one:

<pre class="lang:js decode:true ">import { Directive, provide } from '@angular/core';
import { IntegerValidator } from './integer.validator';
import { NG_VALIDATORS } from '@angular/common';

@Directive({
    selector: '[isInRange][ngControl]',
    providers: [provide(NG_VALIDATORS, { useValue: IntegerValidator.range, multi: true })]
})

export class IsInRangeValidatorDirective {
}</pre>

Here we make the Range-validator accessible to the form by naming it &#8220;isInRange&#8221;. THen we use the angular given NG_Validators and &#8220;extend&#8221; it with our custom validator.

Now we have to tell our form component to provide this validator to our form:

<pre class="lang:c# decode:true">import { Component } from '@angular/core';
import { CORE_DIRECTIVES } from '@angular/common';
import { FoodItem } from './model';
import { IsInRangeValidatorDirective } from './integer.range.validator.directive';

@Component({
    selector: 'foodForm-component',
    directives: [CORE_DIRECTIVES, IsInRangeValidatorDirective],
    templateUrl: 'app/myForm.component.html'
})

export class FoodFormComponent {

    public foodItem: FoodItem;

    constructor() {
        this.foodItem = new FoodItem();
    }

    private AddFood = (): void =&gt; {
        alert("Added " + JSON.stringify(this.foodItem));
    }
}</pre>

In our Formcomponent we use the directive &#8220;IsInRangeValidatorDirective&#8221; we just created.

Now it would be nice to show specific error messages to the users. So we have to ask through the form-variable we introduced and through the control name for the state and if there is a range error.

<pre class="lang:c# decode:true">&lt;form #myForm="ngForm"&gt;
    &lt;div class="form-group"&gt;
        &lt;label for="inputCalories"&gt;Calories&lt;/label&gt;
        &lt;input type="number" class="form-control" id="inputCalories" ngControl="calories" placeholder="Calories" [(ngModel)]="foodItem.Calories"
            isInRange&gt;

        &lt;div *ngIf="myForm?.controls?.calories?.hasError('isInRange') && myForm?.controls?.calories?.touched"&gt;
            Please enter a number in a valid range
        &lt;/div&gt;

    &lt;/div&gt;
    &lt;button type="button" class="btn btn-default" (click)="AddFood()" [disabled]="!myForm?.form.valid"&gt;Submit&lt;/button&gt;
&lt;/form&gt;</pre>

And that&#8217;s basically it.

[<img class="aligncenter size-full wp-image-1283" src="{{site.baseurl}}assets/images/Angular2CustomValidators.jpg" alt="Angular2CustomValidators" width="409" height="261" srcset="{{site.baseurl}}assets/images/Angular2CustomValidators.jpg 409w, {{site.baseurl}}assets/images/Angular2CustomValidators-300x191.jpg 300w" sizes="(max-width: 409px) 100vw, 409px" />]({{site.baseurl}}assets/images/Angular2CustomValidators.jpg)

HTH

Regards

&nbsp;

<https://github.com/FabianGosebrink/Angular1-And-Angular2-Tutorials-With-Demo-App/tree/master/Angular2-Tutorial/05_Forms/end>

&nbsp;

<http://blog.thoughtram.io/angular/2016/03/14/custom-validators-in-angular-2.html>

&nbsp;

Thanks to [Jürgen Gutsch](http://www.gutsch-online.de/) for doing a review of this blogpost.

&nbsp;

Fabian