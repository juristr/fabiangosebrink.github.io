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

In the last 

In the last blogpost I lost a few words about the REST-API the FoodChooser is talking to. In this blog I want to talk about the clients I developed.

Well I started coding an AngularJs 1.x Client with Typecript which is available here

<https://github.com/FabianGosebrink/ASPNET-Foodchooser-Cross-Platform-Angular1>

Of course I could not stop looking and digging into Angular2. So I wrote this client too which is available here:

<https://github.com/FabianGosebrink/ASPNET-Foodchooser-Cross-Platform-Angular2>

Inside this blog I want to loose a few words about how I wrote them what the pitfalls were and how I came up with this solution.

> I will only show Angular2 examples here because it's the only client which is still maintained

### Architecture

The application is divieded into several components with its child components. Due to the fact that the application is not that big at all there is no huge hierarchy.

![Angular2, ASP.NET WebAPI, Azure, Cordova, Cross Platform – My Private Hackathon]({{site.baseurl}}assets/articles/2016-04-26/2c50b8f4-8132-46c4-b632-0488a049fef3.png)

So the only interesting thing is the Food-Component which has two child Components "FoodForm" and "FoodList".

```
@Component({
    selector: 'food-component',
    directives: [ROUTER_DIRECTIVES, CORE_DIRECTIVES, FoodListComponent, FoodListFormComponent],
    providers: [FoodDataService, FoodListDataService],
    templateUrl: 'app/components/food/food.component.html'
})
```

```
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
```

The list component itself is not containing the details-view but redirecting to it while iterating through all the foodItems:

```
import { Component, OnInit } from 'angular2/core';
import { CORE_DIRECTIVES } from 'angular2/common';
import { RouteConfig, ROUTER_DIRECTIVES } from 'angular2/router';
import { FoodDataService } from '../../shared/services/food.dataService';
import { FoodListDataService } from '../../shared/services/foodList.dataService';
import { FoodList } from '../../models/foodList';
import { NeedsAuthentication } from '../../decorators/needsAuthentication';

@Component({
    selector: 'foodlists-component',
    directives: [ROUTER_DIRECTIVES, CORE_DIRECTIVES],
    templateUrl: 'app/components/foodlists/foodlists.component.html'
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
```

and the template

```
<div class="row">
    <div class="col-lg-12">
        <h2 class="page-header">Your Lists  <small>{{allLists?.length}}</small></h2>
    </div>

    <div class="col-lg-12">
        <ul class="list-group">
            <a *ngFor="#item of allLists; #i = index" [routerLink]="['/FoodListDetails', {id: item.Id}]">
                <li class="list-group-item">{{item?.Name}}</li>
            </a>
        </ul>
    </div>
</div>
```

### Authentication

The WebAPI is providing a token endpoint to get tokens from after the login process. I do use a "CurrentUserService" to save this token in the storage and read it again.

```
import { Injectable } from 'angular2/core';
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
```

With this I can read if the user is authenticated in a very basic way.

Further I took a decorator to hook into the creation of components to check if the user is authenticated or not. If not the decorator will redirect to the login page:

```
import { CanActivate, ComponentInstruction, Router} from 'angular2/router';
import { Injector } from 'angular2/core';
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
```

With every request I have to prepare the header which I do in a wrapped Http service.

<https://github.com/FabianGosebrink/ASPNET-Foodchooser-Cross-Platform-Angular2/blob/master/Client/app/shared/services/httpWrapper.service.ts>

Sneak peek:

```
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
```

So I check the headers, append a token if available, set the content-type and accept-properties and give the options back to use it in the REST-Call.

### Cross Platform with Gulp

To give this whole thing a go as an exe and as an app on mobile devices I used cordova and electron with gulp as a taskrunner.

I seperated all the files in the tasks for "electron", "cordova" and "web". In the main gulp file I am just gathering all the information and point the default task only to list all available tasks to _not- start something the developer does not know when he only types "gulp" without a specific command.

```
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
```

For example here is the electron gulp file, which turns this application into an exe

```
gulp.task('build:electron:prod', function(done) {
    runSeq(
        'electron-clean-temp',
        'electron-copy-index-to-temp-folder',
        'electron-copy-images-to-temp-folder',
        'electron-copy-css-to-temp-folder',
        'electron-copy-js-to-temp-folder',
        'electron-copy-app',
        'electron-copy-files-as-is-to-temp-folder',
        'electron-concat-uglify-and-copy-vendor-scripts',
        'electron-inject-js-in-html',
        'electron-inject-css-in-html',
        'electron-copy-assets-to-temp-folder',
        'electron-build-win',
        done);
});
```

For cordova

```
gulp.task('build:apps', function(done) {
    runSeq(
        'cordova-clean-temp',
        'cordova-copy-config-to-temp',
        'cordova-copy-index-to-temp-folder',
        'cordova-copy-images-to-temp-folder',
        'cordova-copy-files-as-is-to-temp-folder',
        'cordova-copy-css-to-temp-folder',
        'cordova-copy-app',
        'cordova-concat-uglify-and-copy-vendor-scripts',
        'cordova-inject-js-in-html',
        'cordova-inject-css-in-html',
        'cordova-build-windows',
        'cordova-build-android',
        'cordova-copy-to-dist',
        done);
});
```

### Conclusion:

I hope with this blog posts I gave you an idea and an introduction in what you can achieve with javascript and ASP.NET WebAPI. Having all the tools like Cordova, gulp. you can build real cross-platform applications.

Hope you liked reading it as i liked making it 🙂

HTH

Regards

Fabian

### Links

<http://blog.thoughtram.io/angular/2015/05/03/the-difference-between-annotations-and-decorators.html>

<https://github.com/thinktecture/boardz-cross-platform-sample>

<https://www.xplatform.rocks/2016/02/14/angular2-and-electron-the-definitive-guide/>
