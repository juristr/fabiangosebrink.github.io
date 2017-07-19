---
title: Exploring the new angular HTTP interface
date: 2017-07-19 19:55
author: Fabian Gosebrink
layout: post
tags: angular http REST
logo: 'assets/images/logo_small.png'
navigation: true
cover: 'assets/images/aerial-view-of-laptop-and-notebook_bw_osc.jpg'
subclass: 'post tag-speeches'
disqus: true
categories: blog articles
---

In this blogpost I want to explore the latest HTTP interface from angular which was introduced in Angular 4.3.

We all need to get our data from any source, mostly this is done via HTTP and any REST backend (like node or ASP.NET Core etc.) or even from files in the *.json format. However, this was possible ever since and is one of the key features of an SPA but Angular introduced a improved version of the HTTP Api in version 4.3. Let's take a look into this one.

## Imports / Module

The new module is provided via the `HttpClientModule` interface. So you have to import it first in your module.

Change

``` import { HttpModule } from '@angular/http'; ```

to

``` import { HttpClientModule } from '@angular/common/http'; ```

```
// ...
import { HttpClientModule } from '@angular/common/http';
// ...

@NgModule({
    imports: [
        // ...
        HttpClientModule,
        // ...
    ],

    declarations: [ ... ],

    providers: [ ... ],

    exports: [ ... ]
})

export class HomeModule { }
```

## Dependency Injection

Now you wil notice that you do not have a "provider for Http" anymore. So we have to change that as well:

```constructor(private http: Http) { }```

```constructor(private http: HttpClient) { }```