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

Hey,

in this blogpost I want to show you how to write custom validators for forms in Angular2.

> Note: While writing this post the current version of Angular is RC1. So all this examples are written with code using the RC1 version

When dealing with forms in Angular2 you can use the build-in validators like: required, minLength or maxLength, pattern&#8230; . Sooner or later you need a custom validator fulfilling a special validation.

Let's have a form first:

```
<form #myForm="ngForm">
    <div class="form-group">
        <label for="inputCalories">Calories</label>
        <input type="number" 
            class="form-control" 
            id="inputCalories" 
            ngControl="calories" 
            placeholder="Calories" 
            [(ngModel)]="foodItem.Calories">
    </div>
    <button type="button" class="btn btn-default" (click)="AddFood()" [disabled]="!myForm?.form.valid">Submit</button>
</form>
```

We see a form which has a variable "myForm" with which we can ask for the current state of the form.

Now we want to write a custom validator checking if the entered number is in a specific range for example.

```
import { Control } from '@angular/common';

export class IntegerValidator {

    static range(c: Control): any {

        if (c.value > 2147483647 || c.value < 0) {
            return { 'isInRange': false };
        }
        return null;
    }
}
```

This validator is named "IntegerValidator" and has a function with a control as a parameter. It returns null if everything is okay and not null if the entered number is not in integer range. You can access the control's value via [control].value.

If we want to use it in the form like

```
<form #myForm="ngForm">
    <div class="form-group">
        <label for="inputCalories">Calories</label>
        <input type="number" 
            class="form-control" 
            id="inputCalories" 
            ngControl="calories" 
            placeholder="Calories" 
            [(ngModel)]="foodItem.Calories"
            isInRange>
    </div>
    <button type="button" class="btn btn-default" (click)="AddFood()" [disabled]="!myForm?.form.valid">Submit</button>
</form>
```

We need a directive to access the validator this way. Let's write one:

```
import { Directive, provide } from '@angular/core';
import { IntegerValidator } from './integer.validator';
import { NG_VALIDATORS } from '@angular/common';

@Directive({
    selector: '[isInRange][ngControl]',
    providers: [provide(NG_VALIDATORS, { useValue: IntegerValidator.range, multi: true })]
})

export class IsInRangeValidatorDirective {
}
```

Here we make the Range-validator accessible to the form by naming it "isInRange". THen we use the angular given NG_Validators and "extend" it with our custom validator.

Now we have to tell our form component to provide this validator to our form:

```
import { Component } from '@angular/core';
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

    private AddFood = (): void => {
        alert("Added " + JSON.stringify(this.foodItem));
    }
}
```

In our Formcomponent we use the directive "IsInRangeValidatorDirective" we just created.

Now it would be nice to show specific error messages to the users. So we have to ask through the form-variable we introduced and through the control name for the state and if there is a range error.

```
<form #myForm="ngForm">
    <div class="form-group">
        <label for="inputCalories">Calories</label>
        <input type="number" class="form-control" id="inputCalories" ngControl="calories" placeholder="Calories" [(ngModel)]="foodItem.Calories"
            isInRange>

        <div *ngIf="myForm?.controls?.calories?.hasError('isInRange') && myForm?.controls?.calories?.touched">
            Please enter a number in a valid range
        </div>

    </div>
    <button type="button" class="btn btn-default" (click)="AddFood()" [disabled]="!myForm?.form.valid">Submit</button>
</form>
```

And that's basically it.

![Angular2CustomValidators]({{site.baseurl}}assets/articles/2016-05-10/5eac1264-a787-43be-9144-58fc0fec9ef4.jpg)

HTH

Regards

<https://github.com/FabianGosebrink/Angular1-And-Angular2-Tutorials-With-Demo-App/tree/master/Angular2-Tutorial/05_Forms/end>

<http://blog.thoughtram.io/angular/2016/03/14/custom-validators-in-angular-2.html>

Thanks to [Jürgen Gutsch](http://www.gutsch-online.de/) for doing a review of this blogpost.

Fabian
