---
title: Consuming a REST API with Angular Http-Service in Typescript
date: 2016-02-01 12:23
author: Fabian Gosebrink
layout: post
tags: Angular REST Typescript
logo: 'assets/images/logo_small.png'
navigation: True
cover: 'assets/images/aerial-view-of-laptop-notebook-mobile-phone-and-coffee-cup-on-wooden-table.jpg'
subclass: 'post tag-speeches'
disqus: true
categories: blog articles
---

**Updated to new syntax**

Hey,

with this blog pot I want to show you how to create a Dataservice to consume (not only) an ASP.NET REST API with the angular2 http module.

In my recent blog post [How to start with Visual Studio (Code), Angular2 beta 0, Typescript &amp; ASP.NET Core 1.0](http://offering.solutions/articles/asp-net/how-to-start-with-visual-studio-code-angular2-beta-0-typescript-asp-net-core-1-0/) I have already mentioned how to start and to install the complete angular2 environment with corresponding tools.

Now I want to show you an example dataservice to call your favourite API.

### Configuration

Its always a good thing if you have your configuration seperated stored anywhere in your application. I always go for a file like "app.constants.ts" where I store all my values. If anything changes there, like a version of the api which is stored in the url or the endpoint/server whatever, I can do those changes immediatelly at one point.


{% highlight ts %}
import { Injectable } from '@angular/core';

@Injectable()
export class Configuration {
    public Server: string = "http://localhost:5000/";
    public ApiUrl: string = "api/";
    public ServerWithApiUrl = this.Server + this.ApiUrl;
}
{% endhighlight %}

Notice the injectable attribute to generate the metadata to make the service available through DI in other modules.

>You can read more about DI in anuglar 2 in this blog post [Dependency Injection in Angular 2](http://blog.thoughtram.io/angular/2015/05/18/dependency-injection-in-angular-2.html)

Now we have this going we can generate our service:

### The Service

First of all you have to create a module which only contains a service which is only responsible for calling an API with a specific endpoint.

{% highlight ts %}
import { Injectable } from '@angular/core';
import { Http, Response, Headers } from '@angular/http';
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

    public GetAll = (): Observable<MyTypedItem[]> => {
        return this._http.get(this.actionUrl)
            .map((response: Response) => <MyTypedItem[]>response.json())
            .catch(this.handleError);
    }

    public GetSingle = (id: number): Observable<MyTypedItem> => {
        return this._http.get(this.actionUrl + id)
            .map((response: Response) => <MyTypedItem>response.json())
            .catch(this.handleError);
    }

    public Add = (itemName: string): Observable<MyTypedItem> => {
        let toAdd = JSON.stringify({ ItemName: itemName });

        return this._http.post(this.actionUrl, toAdd, { headers: this.headers })
            .map((response: Response) => <MyTypedItem>response.json())
            .catch(this.handleError);
    }

    public Update = (id: number, itemToUpdate: MyTypedItem): Observable<MyTypedItem> => {
        return this._http.put(this.actionUrl + id, JSON.stringify(itemToUpdate), { headers: this.headers })
            .map((response: Response) => <MyTypedItem>response.json())
            .catch(this.handleError);
    }

    public Delete = (id: number): Observable<Response> => {
        return this._http.delete(this.actionUrl + id)
            .catch(this.handleError);
    }

    private handleError(error: Response) {
        console.error(error);
        return Observable.throw(error.json().error || 'Server error');
    }
}
{% endhighlight %}

This dataservice gets the configuration we just did and the HTTP-Service via DI. We included it over the new module-loading-syntax. Also notice the typed items we included and the configuration we have to pull in to make it available.

It is also important to tell the http-calls which header to use. "Application/Json" in this case.

Now you can include, inject and use this service to make http-calls to your API like this:

{% highlight ts %}
import { Component, OnInit } from '@angular/core';
import { CORE_DIRECTIVES } from '@angular/common';
import { DataService } from '../services/dataService';
import { MyTypedItem } from '../models/MyTypedItem ';

@Component({
    selector: 'my-item-component',
    providers: [DataService],
    templateUrl: 'app/myItem/myItem.component.html'
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
            .subscribe((data:MyTypedItem[]) => this.myItems = data,
                error => console.log(error),
                () => console.log('Get all Items complete'));
    }
}
{% endhighlight %}

I think this should be basically it. Pay attention to the typed answer you get from the service

```(response:MyTypedItem[]) =>```

and to the subsribe after calling the "GetAll"-Method from the service.

Hope you enjoyed it and a lot more: I hope this helps.

Bye

Fabian

### GitHub:

[Angular 2 / ASP.NET CORE 1.0 Template With SignalR](https://github.com/FabianGosebrink/Angular2-ASPNETCore-SignalR-Demo)

or

[https://github.com/FabianGosebrink/ASPNET-ASPNET-Core-Angular1-Angular2-Demo/tree/master/Angular2-Client](https://github.com/FabianGosebrink/ASPNET-ASPNET-Core-Angular1-Angular2-Demo/tree/master/Angular2-Client)

### Links:

[https://auth0.com/blog/2015/05/14/creating-your-first-real-world-angular-2-app-from-authentication-to-calling-an-api-and-everything-in-between/](https://auth0.com/blog/2015/05/14/creating-your-first-real-world-angular-2-app-from-authentication-to-calling-an-api-and-everything-in-between/)

[https://auth0.com/blog/2015/10/15/angular-2-series-part-3-using-http/](https://auth0.com/blog/2015/10/15/angular-2-series-part-3-using-http/)

[https://auth0.com/blog/2015/09/17/angular-2-series-part-2-domain-models-and-dependency-injection/](https://auth0.com/blog/2015/09/17/angular-2-series-part-2-domain-models-and-dependency-injection/)

[https://auth0.com/blog/2015/09/03/angular2-series-working-with-pipes/](https://auth0.com/blog/2015/09/03/angular2-series-working-with-pipes/)

[https://angular.io/docs/ts/latest/guide/architecture.html](https://angular.io/docs/ts/latest/guide/architecture.html)
