---
title: Angular - New HTTP interface with interceptors
date: 2017-07-19 19:55
author: Fabian Gosebrink
layout: post
tags: angular http REST
logo: 'assets/images/logo_small.png'
navigation: true
cover: 'assets/images/aerial-view-of-laptop-and-notebook_bw_osc.jpg'
subclass: 'post tag-speeches'
disqus: true
categories: articles
---

In this blogpost I want to explore the latest HTTP interface from angular which was introduced in Angular 4.3.

We all need to get our data from any source, mostly this is done via HTTP and any REST backend (like node or ASP.NET Core etc.) or even from files in the *.json format. However, this was possible ever since and is one of the key features of an SPA but Angular introduced a improved version of the HTTP Api in version 4.3. Let's take a look into this one.

## Imports // Module

The new module is provided via the `HttpClientModule` interface. So you have to import it first in your module.

Change

``` import { HttpModule } from '@angular/http'; ```

to

``` import { HttpClientModule } from '@angular/common/http'; ```

{% highlight js %}
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
{% endhighlight %}

## Dependency Injection // Constructors

Now you wil notice that you do not have a "provider for Http" anymore. So we have to change that as well:

So everywhere you inject

```constructor(private http: Http) { }```

You have to change that to

```
import { HttpClient } from '@angular/common/http';
// ...
constructor(private http: HttpClient) { }
```

## Methods // HTTP Verbs

We can use the HTTP methods around GET, POST etc. in a new way. The `.json()` is not necessary anymore. JSON is now assumed as a standard. So that means, we can get rid of the first `.map((response: Response) => <MyType>response.json())` method. In addition to that the method now are generic which means that they can take the type directly and you do not need to cast it anymore. So from

{% highlight js %}
public get(): Observable<MyType> => {
    return this.http.get(url)
        .map((response: Response) => <MyType>response.json());
}
{% endhighlight %}

to

{% highlight js %}
get<T>(url: string): Observable<T> {
    return this.http.get<T>(url);
}
{% endhighlight %}

or for a post request its pretty much straight forward as well

{% highlight js %}
post<T>(url: string, body: string): Observable<T> {
    return this.http.post<T>(url, body);
}
{% endhighlight %}

## Headers

To apply headers we can simply add another parameter to the get function passing an object with a headers property.

So

{% highlight js %}
get<T>(url: string): Observable<T> {
    return this.http.get<T>(url);
}

post<T>(url: string, body: string): Observable<T> {
    return this.http.post<T>(url, body);
}
{% endhighlight %}

becomes to

{% highlight js %}
get<T>(url: string, headers?: HttpHeaders | null): Observable<T> {
    const expandedHeaders = this.prepareHeader(headers);
    return this.http.get<T>(url, expandedHeaders);
}

post<T>(url: string, body: string, headers?: HttpHeaders | null): Observable<T> {
    const expandedHeaders = this.prepareHeader(headers);
    return this.http.post<T>(url, body, expandedHeaders);
}

//...

private prepareHeader(headers: HttpHeaders | null): object {
    headers = headers || new HttpHeaders();

    headers = headers.set('Content-Type', 'application/json');
    headers = headers.set('Accept', 'application/json');

    return {
        headers: headers
    }
}
{% endhighlight %}

Be aware that setting a new header _returns_ the new header object instead of manipulation your existing one. Its immutable.

## Interceptors

From AngularJS we know HTTP interceptors as a great and very mighty way to observe ingoing and outgoing requests. In Angular this feature of interceptors was not available yet. But it came back with this new version of the http interface.

Let us add an interceptor which is just logging the requests. This is just a class tagged with the `@Injectable` decorator implementing and interface `HttpInterceptor`. It gets passed an `HttpHeader` which provides us the `handle(...)` method to not cancel the request but just hook 'between' the application and the outgoing or incoming request. So we have to give the request further to be handled from the enxt interceptor (chaining) or the backend.

{% highlight js %}
import { Injectable} from '@angular/core';
import { HttpEvent, HttpInterceptor, HttpHandler, HttpRequest } from '@angular/common/http';

@Injectable()
export class MyFirstInterceptor implements HttpInterceptor {
    intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
        console.log(req);
        return next.handle(req);
    }
}
{% endhighlight %}

and of course we have ot reigster it on the module

{% highlight js %}
providers: [
    // ...
    {
        provide: HTTP_INTERCEPTORS,
        useClass: MyFirstInterceptor,
        multi: true,
    }]
{% endhighlight %}

If we just log this one we get a result like this

{% highlight js %}
{
	"url": "http://blabla",
	"body": null,
	"reportProgress": false,
	"withCredentials": false,
	"responseType": "json",
	"method": "GET",
	"headers": {
		"normalizedNames": {},
		"lazyUpdate": null,
		"headers": {}
	},
	"params": {
		"updates": null,
		"cloneFrom": null,
		"encoder": {},
		"map": null
	},
	"urlWithParams": "http://blablawithparams"
}
{% endhighlight %}

Now we can tweak the interceptor to add the headers accordingly and add the authentication token if the user has one.

{% highlight js %}
@Injectable()
export class MyFirstInterceptor implements HttpInterceptor {

    constructor(private currentUserService: CurrentUserService) { }

    intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {

        // get the token from a service
        const token: string = this.currentUserService.token;

        // add it if we have one
        if (token) {
            req = req.clone({ headers: req.headers.set('Authorization', 'Bearer ' + token) });
        }

        // if this is a login-request the header is 
        // already set to x/www/formurl/encoded. 
        // so if we already have a content-type, do not 
        // set it, but if we don't have one, set it to 
        // default --> json
        if (!req.headers.has('Content-Type')) {
            req = req.clone({ headers: req.headers.set('Content-Type', 'application/json') });
        }

        // setting the accept header
        req = req.clone({ headers: req.headers.set('Accept', 'application/json') });
        return next.handle(req);
    }
}
{% endhighlight %}

With this we can get rid of the method in the class above and we dont need to call the method `private prepareHeader(headers: HttpHeaders | null): object` to prepare the headers. So the final result could be like:

{% highlight js %}
import { HttpClient, HttpHeaders } from '@angular/common/http';
import { HttpEvent, HttpHandler, HttpInterceptor, HttpRequest } from '@angular/common/http';
import { Injectable } from '@angular/core';
import { Observable } from 'rxjs/Observable';

import { CurrentUserService } from '../services/currentUser.service';

@Injectable()
export class MyDataService {
    constructor(private http: HttpClient, private currentUserService: CurrentUserService) { }

    get<T>(url: string): Observable<T> {
        return this.http.get<T>(url);
    }

    post<T>(url: string, body: string): Observable<T> {
        return this.http.post<T>(url, body);
    }

    put<T>(url: string, body: string): Observable<T> {
        return this.http.put<T>(url, body);
    }

    delete<T>(url: string): Observable<T> {
        return this.http.delete<T>(url);
    }

    patch<T>(url: string, body: string): Observable<T> {
        return this.http.patch<T>(url, body);
    }
}


@Injectable()
export class MyFirstInterceptor implements HttpInterceptor {

    constructor(private currentUserService: CurrentUserService) { }

    intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
        console.log(JSON.stringify(req));

        const token: string = this.currentUserService.token;

        if (token) {
            req = req.clone({ headers: req.headers.set('Authorization', 'Bearer ' + token) });
        }

        if (!req.headers.has('Content-Type')) {
            req = req.clone({ headers: req.headers.set('Content-Type', 'application/json') });
        }

        req = req.clone({ headers: req.headers.set('Accept', 'application/json') });
        return next.handle(req);
    }
}

{% endhighlight %}

I hope with this I got you a basic understanding of the new neat HTTP api from Angular. I like it a lot and this code looks nicer and cleaner that way.

BR

Fabian