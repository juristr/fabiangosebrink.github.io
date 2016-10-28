---
id: 1234
title: 'Angular2, ASP.NET WebAPI, Azure, Cordova, Cross Platform – My Private Hackathon Part 2'
date: 2016-04-26T10:05:19+00:00
author: Fabian Gosebrink
layout: post
tags: angular2 asp.net azure cordova crossplatform gulp webapi 
logo: 'assets/images/logo_small.png'
navigation: True
cover: 'assets/images/download_edit_dark.jpg'
subclass: 'post tag-speeches'
disqus: true
categories: articles
---

In the last <a href="http://offering.solutions/articles/asp-net/angular2-asp-net-webapi-azure-my-private-hackathon/">blogpost </a>I lost a few words about the REST-API the FoodChooser is talking to. In this blog I want to talk about the clients I developed.

Well I started coding an AngularJs 1.x Client with Typecript which is available here

<a href="https://github.com/FabianGosebrink/ASPNET-Foodchooser-Cross-Platform-Angular1">https://github.com/FabianGosebrink/ASPNET-Foodchooser-Cross-Platform-Angular1</a>

Of course I could not stop looking and digging into Angular2. So I wrote this client too which is available here:

<a href="https://github.com/FabianGosebrink/ASPNET-Foodchooser-Cross-Platform-Angular2">https://github.com/FabianGosebrink/ASPNET-Foodchooser-Cross-Platform-Angular2</a>

&nbsp;

Inside this blog I want to loose a few words about how I wrote them what the pitfalls were and how I came up with this solution.

<blockquote>I will only show Angular2 examples here because it's the only client which is still maintained</blockquote>

<h3>Architecture</h3>

The application is divieded into several components with its child components. Due to the fact that the application is not that big at all there is no huge hierarchy.

<a href="http://offering.solutions/wp-content/uploads/2016/04/SiteMap.png"><img class="aligncenter wp-image-1243 size-full" src="http://offering.solutions/wp-content/uploads/2016/04/SiteMap.png" alt="Angular 2, ASP.NET WebAPI, Azure &amp; Cordova, Cross Platform" width="739" height="377" /></a>

So the only interesting thing is the Food-Component which has two child Components "FoodForm" and "FoodList".

<pre class="lang:js decode:true">@Component({
    selector: 'food-component',
    directives: [ROUTER_DIRECTIVES, CORE_DIRECTIVES, FoodListComponent, FoodListFormComponent],
    providers: [FoodDataService, FoodListDataService],
    template: require('./food.component.html')
})</pre>

<pre class="lang:xhtml decode:true ">&lt;!-- Page Content --&gt;
&lt;div class="container"&gt;
    &lt;!-- Introduction Row --&gt;
    &lt;div class="row"&gt;
        &lt;div class="col-lg-12"&gt;
            &lt;h1 class="page-header"&gt;Foodlists
                    &lt;small&gt;See all your food lists&lt;/small&gt;
                &lt;/h1&gt;
        &lt;/div&gt;
    &lt;/div&gt;

    &lt;foodListForm-component&gt;&lt;/foodListForm-component&gt;
    &lt;foodlists-component&gt;&lt;/foodlists-component&gt;

&lt;/div&gt;
&lt;!-- /.container --&gt;</pre>

The list component itself is not containing the details-view but redirecting to it while iterating through all the foodItems:

<pre class="lang:js decode:true">import { Component, OnInit } from '@angular/core';
import { CORE_DIRECTIVES } from '@angular/common';
import { RouteConfig, ROUTER_DIRECTIVES } from '@angular/router-deprecated';
import { FoodDataService } from '../../shared/services/food.dataService';
import { FoodListDataService } from '../../shared/services/foodList.dataService';
import { FoodList } from '../../models/FoodList';
import { NeedsAuthentication } from '../../decorators/needsAuthentication';

@Component({
    selector: 'foodlists-component',
    directives: [ROUTER_DIRECTIVES, CORE_DIRECTIVES],
    template: require('./foodlists.component.html')
})

@NeedsAuthentication()
export class FoodListComponent implements OnInit {

    allLists: FoodList[];

    constructor(private _foodListDataService: FoodListDataService) {
        _foodListDataService.foodListAdded.subscribe((foodList: FoodList) =&gt; {
            this.getAllLists();
        });
    }

    public ngOnInit() {
        this.getAllLists();
    }

    private getAllLists() {
        this._foodListDataService
            .GetAllLists()
            .subscribe((response: FoodList[]) =&gt; {
                this.allLists = response;
                console.log(response.length);
            }, error =&gt; {
                this.errorMessage = error;
            });
    }
}</pre>

and the template

<pre class="lang:js decode:true">&lt;!-- Team Members Row --&gt;
&lt;div class="row"&gt;
    &lt;div class="col-lg-12"&gt;
        &lt;h2 class="page-header"&gt;Your Lists  &lt;small&gt;{{allLists?.length}}&lt;/small&gt;&lt;/h2&gt;
    &lt;/div&gt;

    &lt;div class="col-lg-12"&gt;
        &lt;ul class="list-group"&gt;
            &lt;a *ngFor="let item of allLists; let i=index" [routerLink]="['/FoodListDetails', {id: item.Id}]"&gt;
                &lt;li class="list-group-item"&gt;{{item?.Name}}&lt;/li&gt;
            &lt;/a&gt;
        &lt;/ul&gt;
    &lt;/div&gt;
&lt;/div&gt;</pre>

<h3>Authentication</h3>

The WebAPI is providing a token endpoint to get tokens from after the login process. I do use a "CurrentUserService" to save this token in the storage and read it again.

<pre class="lang:js decode:true">import { Injectable } from '@angular/core';
import { StorageService } from './storage.service';

@Injectable()
export class CurrentUserService {

    constructor(private _storageService: StorageService) {

    }

    public get token(): string {
        let token = this._storageService.getItem('auth');

        return token;
    }

    public set token(token: string) {
        this._storageService.setItem('auth', token);
    }
}</pre>

With this I can read if the user is authenticated in a very basic way.

Further I took a decorator to hook into the creation of components to check if the user is authenticated or not. If not the decorator will redirect to the login page:

<pre class="lang:js decode:true">import { CanActivate, ComponentInstruction, Router} from '@angular/router-deprecated';
import { Injector } from '@angular/core';
import { appInjector } from '../shared/services/appInjector';
import { StorageService } from '../shared/services/storage.service';

export const NeedsAuthentication = () =&gt; {
    return CanActivate((to: ComponentInstruction, from: ComponentInstruction, target = ['/']) =&gt; {
        let injector: Injector = appInjector();
        let router: Router = injector.get(Router);
        let storageService: StorageService = injector.get(StorageService);

        if (storageService.getItem('auth')) {
            return true;
        }

        router.navigate(['/Login', { target }]);

        return false;
    });
}</pre>

With every request I have to prepare the header which I do in a wrapped Http service.

<a href="https://github.com/FabianGosebrink/ASPNET-Foodchooser-Cross-Platform-Angular2/blob/master/Client/app/shared/services/httpWrapper.service.ts">https://github.com/FabianGosebrink/ASPNET-Foodchooser-Cross-Platform-Angular2/blob/master/Client/app/shared/services/httpWrapper.service.ts</a>

Sneak peek:

&nbsp;

<pre class="lang:js decode:true "> private prepareOptions(options: RequestOptionsArgs): RequestOptionsArgs {
        let token: string = this._currentUserService.token;

        options = options || {};

        if (!options.headers) {
            options.headers = new Headers();
        }

        if (token) {
            options.headers.append('Authorization', 'Bearer ' + token);
        }

        options.headers.append('Content-Type', 'application/json');
        options.headers.append('Accept', 'application/json');

        return options;
    }</pre>

So I check the headers, append a token if available, set the content-type and accept-properties and give the options back to use it in the REST-Call.

&nbsp;

<h3>Cross Platform with Gulp</h3>

To give this whole thing a go as an exe and as an app on mobile devices I used cordova and electron with gulp as a taskrunner.

I seperated all the files in the tasks for "electron", "cordova" and "web". In the main gulp file I am just gathering all the information and point the default task only to list all available tasks to _not- start something the developer does not know when he only types "gulp" without a specific command.

<pre class="lang:js decode:true ">var buildConfig = require('./gulp.config');

gulp.task('default', ['help']);
gulp.task('help', taskListing.withFilters(/-/));

require('./gulpTasks/web');
require('./gulpTasks/electron');
require('./gulpTasks/cordova');

gulp.task('build:all', function(done) {
    runSeq(
        'build:web:prod',
        'build:electron:prod',
        'build:apps',
        done);
});</pre>

For example here is the electron gulp file, which turns this application into an exe

&nbsp;

<pre class="lang:js decode:true">gulp.task('build:electron:prod', function (done) {
    runSeq(
        'electron-clean-temp',
        'electron-compile-with-webpack',
        'electron-copy-index-to-temp-folder',
        'electron-inject-in-html',
        'electron-copy-assets-to-temp-folder',
        'electron-build-win',
        done);
});</pre>

For cordova

<pre class="lang:js decode:true ">gulp.task('build:apps', function (done) {
    runSeq(
        'cordova-clean-temp',
        'cordova-copy-config-to-temp',
        'cordova-copy-winstore-to-temp',
        'cordova-copy-index-to-temp-folder',
        'cordova-copy-images-to-temp-folder',
        'cordova-compile-with-webpack',
        'cordova-inject-in-html',
        'cordova-build-windows',
        //  'cordova-build-android',
        'cordova-clean-dist',
        'cordova-copy-to-dist',
        done);
});</pre>

<h3></h3>

<h3>Conclusion:</h3>

I hope with this blog posts I gave you an idea and an introduction in what you can achieve with javascript and ASP.NET WebAPI. Having all the tools like Cordova, gulp. you can build real cross-platform applications.

Hope you liked reading it as i liked making it :)

HTH

Regards

Fabian

<h3></h3>

<h3>Links</h3>

<a href="http://blog.thoughtram.io/angular/2015/05/03/the-difference-between-annotations-and-decorators.html">http://blog.thoughtram.io/angular/2015/05/03/the-difference-between-annotations-and-decorators.html</a>

<a href="https://github.com/thinktecture/boardz-cross-platform-sample">https://github.com/thinktecture/boardz-cross-platform-sample</a>

<a href="https://www.xplatform.rocks/2016/02/14/angular2-and-electron-the-definitive-guide/">https://www.xplatform.rocks/2016/02/14/angular2-and-electron-the-definitive-guide/</a>

&nbsp;

&nbsp;
