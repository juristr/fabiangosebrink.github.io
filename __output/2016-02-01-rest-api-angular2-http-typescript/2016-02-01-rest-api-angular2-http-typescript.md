---
id: 1032
title: Consuming a REST API with Angular2 Http-Service in Typescript
date: 2016-02-01T12:23:46+00:00
author: Fabian Gosebrink
layout: post
tags: angular2;typescript 
logo: 'assets/images/logo_small.png'
navigation: True
cover: 'assets/images/download_edit_dark.jpg'
subclass: 'post tag-speeches'
---

# Consuming a REST API with Angular2 Http-Service in Typescript

Hey,

with this blog pot I want to show you how to create a Dataservice to consume (not only) an ASP.NET REST API with the angular2 http module.

In my recent blog post [How to start with Visual Studio (Code), Angular2 beta 0, Typescript & ASP.NET Core 1.0](http://offering.solutions/articles/asp-net/how-to-start-with-visual-studio-code-angular2-beta-0-typescript-asp-net-core-1-0/) I have already mentioned how to start and to install the complete angular2 environment with corresponding tools.

Now I want to show you an example dataservice to call your favourite API.

### Configuration

Its always a good thing if you have your configuration seperated stored anywhere in your application. I always go for a file like &#8220;app.constants.ts&#8221; where I store all my values. If anything changes there, like a version of the api which is stored in the url or the endpoint/server whatever, I can do those changes immediatelly at one point.

&nbsp;

<pre class="lang:c# decode:true " title="app.constants.ts">import { Injectable } from 'angular2/core';

@Injectable()
export class Configuration {
    public Server: string = "http://localhost:5000/";
    public ApiUrl: string = "api/";
    public ServerWithApiUrl = this.Server + this.ApiUrl;
}</pre>

Notice the injectable attribute to generate the metadata to make the service available through DI in other modules.

> You can read more about DI in anuglar 2 in this blog post [Dependency Injection in Angular 2](http://blog.thoughtram.io/angular/2015/05/18/dependency-injection-in-angular-2.html)

Now we have this going we can generate our service:

### The Service

First of all you have to create a module which only contains a service which is only responsible for calling an API with a specific endpoint.

<pre class="lang:c# decode:true ">import { Injectable } from 'angular2/core';
import { Http, Response, Headers } from 'angular2/http';
import 'rxjs/add/operator/map'
import { Observable } from 'rxjs/Observable';
import { MyTypedItem } from '../models/MyTypedItem';
import { Configuration } from '../app.constants';

@Injectable()
export class DataService {

    private actionUrl: string;
    private headers: Headers;

    constructor(private _http: Http, private _configuration: Configuration) {
        
        this.actionUrl = _configuration.ServerWithApiUrl + 'myItem/';
        
        this.headers = new Headers();
        this.headers.append('Content-Type', 'application/json');
        this.headers.append('Accept', 'application/json');
    }

    public GetAll = (): Observable&lt;Response&gt; =&gt; {
        return this._http.get(this.actionUrl).map(res =&gt; res.json());
    }

    public GetSingle = (id: number): Observable&lt;Response&gt; =&gt; {
        return this._http.get(this.actionUrl + id).map(res =&gt; res.json());
    }

    public Add = (itemName: string): Observable&lt;Response&gt; =&gt; {
        var toAdd = JSON.stringify({ ItemName: itemName });

        return this._http.post(this.actionUrl, toAdd, { headers: this.headers }).map(res =&gt; res.json());
    }

    public Update = (id: number, itemToUpdate: MyTypedItem): Observable&lt;Response&gt; =&gt; {
        return this._http.put(this.actionUrl + id, JSON.stringify(itemToUpdate), { headers: this.headers }).map(res =&gt; res.json());
    }

    public Delete = (id: number): Observable&lt;Response&gt; =&gt; {
        return this._http.delete(this.actionUrl + id);
    }
}</pre>

This dataservice gets the configuration we just did and the HTTP-Service via DI. We included it over the new module-loading-syntax. Also notice the typed items we included and the configuration we have to pull in to make it available.

It is also important to tell the http-calls which header to use. &#8220;Application/Json&#8221; in this case.

Now you can include, inject and use this service to make http-calls to your API like this:

<pre class="lang:c# decode:true">import { Component, OnInit } from 'angular2/core';
import { CORE_DIRECTIVES } from 'angular2/common';
import { DataService } from '../services/dataService';
import { MyTypedItem } from '../models/MyTypedItem ';

@Component({
    selector: 'my-item-component',
    providers: [DataService],
    templateUrl: 'app/myItem/myItem.component.html',
    directives: [CORE_DIRECTIVES]
})

export class MyItemComponent implements OnInit {
    public myItems: MyTypedItem [];

    constructor(private _dataService: DataService) { }

    ngOnInit() {
        this.getAllItems();
    }
    
    //...

    private getAllItems(): void {
        this._dataService
            .GetAll()
            .subscribe((data:MyTypedItem[]) =&gt; this.myItems = data,
                error =&gt; console.log(error),
                () =&gt; console.log('Get all Items complete'));
    }
}</pre>

&nbsp;

I think this should be basically it. Pay attention to the typed answer you get from the service

<pre class="lang:c# decode:true">(data:MyTypedItem[]) =&gt;</pre>

and to the subsribe after calling the &#8220;GetAll&#8221;-Method from the service.

Hope you enjoyed it and a lot more: I hope this helps.

&nbsp;

Bye

Fabian

GitHub:

[Angular 2 / ASP.NET CORE 1.0 Template With SignalR](https://github.com/FabianGosebrink/Angular2-ASPNETCore-SignalR-Demo)

&nbsp;

links:

<https://auth0.com/blog/2015/05/14/creating-your-first-real-world-angular-2-app-from-authentication-to-calling-an-api-and-everything-in-between/>

<https://auth0.com/blog/2015/10/15/angular-2-series-part-3-using-http/>

<https://auth0.com/blog/2015/09/17/angular-2-series-part-2-domain-models-and-dependency-injection/>

<https://auth0.com/blog/2015/09/03/angular2-series-working-with-pipes/>

<https://angular.io/docs/ts/latest/guide/architecture.html>
