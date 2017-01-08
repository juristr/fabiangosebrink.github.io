---
title: Angular, ASP.NET WebAPI, Azure & Cordova, Cross Platform – My Private Hackathon Part 2
date: 2016-04-26 10:05
author: Fabian Gosebrink
layout: post
tags: Angular ASP.NET Azure Cordova Cross Platform Gulp WebAPI
logo: 'assets/images/logo_small.png'
navigation: True
cover: 'assets/images/aerial-view-of-laptop-notebook-mobile-phone-and-coffee-cup-on-wooden-table.jpg'
subclass: 'post tag-speeches'
disqus: true
categories: articles
---

In the last [blogpost](http://offering.solutions/articles/asp-net/angular2-asp-net-webapi-azure-my-private-hackathon/)I lost a few words about the REST-API the FoodChooser is talking to. In this blog I want to talk about the clients I developed. This is: Angular, ASP.NET WebAPI, Azure & Cordova, Cross Platform – My Private Hackathon Part 2.

Well I started coding an AngularJs 1.x Client with Typecript which is available here

[https://github.com/FabianGosebrink/ASPNET-Foodchooser-Cross-Platform-Angular1](https://github.com/FabianGosebrink/ASPNET-Foodchooser-Cross-Platform-Angular1)

Of course I could not stop looking and digging into Angular. So I wrote this client too which is available here:

[https://github.com/FabianGosebrink/ASPNET-Foodchooser-Cross-Platform-Angular2](https://github.com/FabianGosebrink/ASPNET-Foodchooser-Cross-Platform-Angular2)

Inside this blog I want to loose a few words about how I wrote them what the pitfalls were and how I came up with this solution.

>I will only show Angular2 examples here because it's the only client which is still maintained

### Architecture

The application is divieded into several components with its child components. Due to the fact that the application is not that big at all there is no huge hierarchy.

![Angular 2, ASP.NET WebAPI, Azure & Cordova, Cross Platform]({{site.baseurl}}assets/articles/wp-content/uploads/2016/04/SiteMap.png)

So the only interesting thing is the Food-Component which has two child Components "FoodForm" and "FoodList".

{% highlight ts %}
@Component({
    selector: 'food-component',
    directives: [ROUTER_DIRECTIVES, CORE_DIRECTIVES, FoodListComponent, FoodListFormComponent],
    providers: [FoodDataService, FoodListDataService],
    template: require('./food.component.html')
})
{% endhighlight %}

{% highlight html %}
<!-- Page Content -->
<div class="container">
    <!-- Introduction Row -->
    <div class="row">
        <div class="col-lg-12">
            <h1 class="page-header">Foodlists
                    <small>See all your food lists</small>
                </h1>
        </div>
    </div>

    <foodListForm-component></foodListForm-component>
    <foodlists-component></foodlists-component>

</div>
{% endhighlight %}

The list component itself is not containing the details-view but redirecting to it while iterating through all the foodItems:

{% highlight ts %}
import { Component, OnInit } from '@angular/core';
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
        _foodListDataService.foodListAdded.subscribe((foodList: FoodList) => {
            this.getAllLists();
        });
    }

    public ngOnInit() {
        this.getAllLists();
    }

    private getAllLists() {
        this._foodListDataService
            .GetAllLists()
            .subscribe((response: FoodList[]) => {
                this.allLists = response;
                console.log(response.length);
            }, error => {
                this.errorMessage = error;
            });
    }
}
{% endhighlight %}

and the template

{% highlight html %}
<!-- Team Members Row -->
<div class="row">
    <div class="col-lg-12">
        <h2 class="page-header">Your Lists  <small>{{allLists?.length}}</small></h2>
    </div>

    <div class="col-lg-12">
        <ul class="list-group">
            <a *ngFor="let item of allLists; let i=index" [routerLink]="['/FoodListDetails', {id: item.Id}]">
                <li class="list-group-item">{{item?.Name}}</li>
            </a>
        </ul>
    </div>
</div>
{% endhighlight %}

### Authentication

The WebAPI is providing a token endpoint to get tokens from after the login process. I do use a "CurrentUserService" to save this token in the storage and read it again.

{% highlight ts %}
import { Injectable } from '@angular/core';
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
}
{% endhighlight %}

With this I can read if the user is authenticated in a very basic way.

Further I took a decorator to hook into the creation of components to check if the user is authenticated or not. If not the decorator will redirect to the login page:

{% highlight ts %}
import { CanActivate, ComponentInstruction, Router} from '@angular/router-deprecated';
import { Injector } from '@angular/core';
import { appInjector } from '../shared/services/appInjector';
import { StorageService } from '../shared/services/storage.service';

export const NeedsAuthentication = () => {
    return CanActivate((to: ComponentInstruction, from: ComponentInstruction, target = ['/']) => {
        let injector: Injector = appInjector();
        let router: Router = injector.get(Router);
        let storageService: StorageService = injector.get(StorageService);

        if (storageService.getItem('auth')) {
            return true;
        }

        router.navigate(['/Login', { target }]);

        return false;
    });
}
{% endhighlight %}

With every request I have to prepare the header which I do in a wrapped Http service.

[https://github.com/FabianGosebrink/ASPNET-Foodchooser-Cross-Platform-Angular2/blob/master/Client/app/shared/services/httpWrapper.service.ts](https://github.com/FabianGosebrink/ASPNET-Foodchooser-Cross-Platform-Angular2/blob/master/Client/app/shared/services/httpWrapper.service.ts)

Sneak peek:


{% highlight ts %}
private prepareOptions(options: RequestOptionsArgs): RequestOptionsArgs {
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
}
{% endhighlight %}

So I check the headers, append a token if available, set the content-type and accept-properties and give the options back to use it in the REST-Call.

### Cross Platform with Gulp

To give this whole thing a go as an exe and as an app on mobile devices I used cordova and electron with gulp as a taskrunner.

I seperated all the files in the tasks for "electron", "cordova" and "web". In the main gulp file I am just gathering all the information and point the default task only to list all available tasks to _not- start something the developer does not know when he only types "gulp" without a specific command.

{% highlight js %}
var buildConfig = require('./gulp.config');

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
});
{% endhighlight %}

For example here is the electron gulp file, which turns this application into an exe

{% highlight js %}
gulp.task('build:electron:prod', function (done) {
    runSeq(
        'electron-clean-temp',
        'electron-compile-with-webpack',
        'electron-copy-index-to-temp-folder',
        'electron-inject-in-html',
        'electron-copy-assets-to-temp-folder',
        'electron-build-win',
        done);
});
{% endhighlight %}

For cordova

{% highlight js %}
gulp.task('build:apps', function (done) {
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
});
{% endhighlight %}


### Conclusion:

I hope with this blog posts I gave you an idea and an introduction in what you can achieve with javascript and ASP.NET WebAPI. Having all the tools like Cordova, gulp. you can build real cross-platform applications.

Hope you liked reading it as i liked making it :)

HTH

Regards

Fabian

### Links

[http://blog.thoughtram.io/angular/2015/05/03/the-difference-between-annotations-and-decorators.html](http://blog.thoughtram.io/angular/2015/05/03/the-difference-between-annotations-and-decorators.html)

[https://github.com/thinktecture/boardz-cross-platform-sample](https://github.com/thinktecture/boardz-cross-platform-sample)

[https://www.xplatform.rocks/2016/02/14/angular2-and-electron-the-definitive-guide/](https://www.xplatform.rocks/2016/02/14/angular2-and-electron-the-definitive-guide/)
