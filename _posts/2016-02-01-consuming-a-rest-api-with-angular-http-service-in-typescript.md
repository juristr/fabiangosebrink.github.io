---
title: Consuming a REST API with Angular Http-Service in Typescript
date: 2016-02-01 12:23
author: Fabian Gosebrink
layout: post
tags: angular restapi typescript
logo: 'assets/images/logo_small.png'
navigation: True
cover: 'assets/images/aerial-view-of-laptop-and-notebook_bw_osc.jpg'
subclass: 'post tag-speeches'
disqus: true
categories: articles
---

### Updates

19.08.2017 - Updated to ASP.NET Core 2.0 & new HttpClientModule

### Blogpost

Hey,

with this blog pot I want to show you how to create a dataservice to consume (not only) an ASP.NET REST API with the angular http module.

In my recent blog post [Getting started with Visual Studio Code, AngularJS and Typescript](http://offering.solutions/blog/articles/2015/12/03/getting-started-with-visual-studio-code-angularjs-and-typescript/) I have already mentioned how to start and to install the complete angular environment with corresponding tools.

Now I want to show you an example dataservice to call your favourite API.

### Configuration

Its always a good thing if you have your configuration seperated stored anywhere in your application. I always go for a file like "app.constants.ts" where I store all my values. If anything changes there, like a version of the api which is stored in the url or the endpoint/server whatever, I can do those changes immediatelly at one point.

{% highlight ts %}
import { Injectable } from '@angular/core';

@Injectable()
export class Configuration {
    public Server = 'http://localhost:5000/';
    public ApiUrl = 'api/';
    public ServerWithApiUrl = this.Server + this.ApiUrl;
}
{% endhighlight %}

Notice the injectable attribute to generate the metadata to make the service available through DI in other modules.

> You can read more about DI in Angular in this blog post [Dependency Injection in Angular](http://blog.thoughtram.io/angular/2015/05/18/dependency-injection-in-angular-2.html)

Now we have this going we can generate our service:

### The Service

First of all you have to create a module which only contains a service which is only responsible for calling an API with a specific endpoint.

{% highlight ts %}
import 'rxjs/add/operator/map';

import { HttpClient, HttpEvent, HttpHandler, HttpInterceptor, HttpRequest } from '@angular/common/http';
import { Injectable } from '@angular/core';
import { Observable } from 'rxjs/Observable';

import { Configuration } from '../../app.constants';

@Injectable()
export class DataService {

    private actionUrl: string;

    constructor(private http: HttpClient, private _configuration: Configuration) {
        this.actionUrl = _configuration.ServerWithApiUrl + 'values/';
    }

    public getAll<T>(): Observable<T> {
        return this.http.get<T>(this.actionUrl);
    }

    public getSingle<T>(id: number): Observable<T> {
        return this.http.get<T>(this.actionUrl + id);
    }

    public add<T>(itemName: string): Observable<T> {
        const toAdd = JSON.stringify({ ItemName: itemName });

        return this.http.post<T>(this.actionUrl, toAdd);
    }

    public update<T>(id: number, itemToUpdate: any): Observable<T> {
        return this.http
            .put<T>(this.actionUrl + id, JSON.stringify(itemToUpdate));
    }

    public delete<T>(id: number): Observable<T> {
        return this.http.delete<T>(this.actionUrl + id);
    }
}


@Injectable()
export class CustomInterceptor implements HttpInterceptor {

    intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
        if (!req.headers.has('Content-Type')) {
            req = req.clone({ headers: req.headers.set('Content-Type', 'application/json') });
        }

        req = req.clone({ headers: req.headers.set('Accept', 'application/json') });
        console.log(JSON.stringify(req.headers));
        return next.handle(req);
    }
}
{% endhighlight %}

This dataservice gets the configuration we just did and the HTTP-Service via DI. We included it over the new module-loading-syntax.

It is also important to tell the http-calls which header to use. "Application/Json" in this case. We are doing this via an interceptor and the new HttpClientModule.

Now you can include, inject and use this service to make http-calls to your API like this:

{% highlight ts %}
import { Component, OnInit } from '@angular/core';
import { ToasterService } from 'angular2-toaster/angular2-toaster';
import { SlimLoadingBarService } from 'ng2-slim-loading-bar';

import { DataService } from '../../shared/services/dataService';

@Component({
    selector: 'app-home-component',
    templateUrl: './home.component.html'
})

export class HomeComponent implements OnInit {

    public message: string;
    public values: any[];

    constructor(
        private _dataService: DataService,
        private _toasterService: ToasterService,
        private _slimLoadingBarService: SlimLoadingBarService) {
        this.message = 'Hello from HomeComponent constructor';
    }

    ngOnInit() {
        this._slimLoadingBarService.start();

        this._dataService
            .getAll<any[]>()
            .subscribe((data: any[]) => this.values = data,
            error => () => {
                this._toasterService.pop('error', 'Damn', 'Something went wrong...');
            },
            () => {
                this._toasterService.pop('success', 'Complete', 'Getting all values complete');
                this._slimLoadingBarService.complete();
            });
    }
}

{% endhighlight %}

I think this should be basically it. Pay attention to the (normally) typed answer you get from the service

```(data: any[]) =>```

should be in your application

{% highlight ts %}
 this._dataService
            .getAll<MyTypedItem[]>()
            .subscribe((data: MyTypedItem[]) => this.values = data,
{% endhighlight %}

and to the subscribe after calling the "GetAll"-Method from the service.

Hope you enjoyed it and a lot more: I hope this helps.

Bye

Fabian

### GitHub:

[Angular // Webpack // ASP.NET CORE WebAPI Starter Template](https://github.com/FabianGosebrink/ASPNETCore-Angular-Webpack-StarterTemplate)

or

[https://github.com/FabianGosebrink/ASPNET-ASPNETCore-Angular-Webpack/tree/master/AngularCLI](https://github.com/FabianGosebrink/ASPNET-ASPNETCore-Angular-Webpack/tree/master/AngularCLI)

### Links:
[https://auth0.com/blog/2015/05/14/creating-your-first-real-world-angular-2-app-from-authentication-to-calling-an-api-and-everything-in-between/](https://auth0.com/blog/2015/05/14/creating-your-first-real-world-angular-2-app-from-authentication-to-calling-an-api-and-everything-in-between/)

[https://auth0.com/blog/2015/10/15/angular-2-series-part-3-using-http/](https://auth0.com/blog/2015/10/15/angular-2-series-part-3-using-http/)

[https://auth0.com/blog/2015/09/17/angular-2-series-part-2-domain-models-and-dependency-injection/](https://auth0.com/blog/2015/09/17/angular-2-series-part-2-domain-models-and-dependency-injection/)

[https://auth0.com/blog/2015/09/03/angular2-series-working-with-pipes/](https://auth0.com/blog/2015/09/03/angular2-series-working-with-pipes/)

[https://angular.io/docs/ts/latest/guide/architecture.html](https://angular.io/docs/ts/latest/guide/architecture.html)
