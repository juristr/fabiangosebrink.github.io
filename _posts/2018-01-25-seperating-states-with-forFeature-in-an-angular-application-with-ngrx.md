---
title: Separating states into modules with ngrx and Angular
date: 2018-01-25 18:40
author: Fabian Gosebrink
layout: post
tags: angular ngrx
logo: 'assets/images/logo_small.png'
navigation: true
cover: 'assets/images/aerial-view-of-laptop-and-notebook_bw_osc.jpg'
subclass: 'post tag-speeches'
disqus: true
categories: articles
---

In this blogpost I want to give you an explanation of the state tree of ngrx if you are working with a state and how to separate it into different modules.

## In this blog

1. [One state on you whole application with forRoot(...)](#forRoot)
2. [Separating state into modules with forFeature(...)](#forFeature)
3. [Conclusion](#conclusion)

If you are building smaller or large angular applications you will sooner or later face the problem to manage the state of your application. It gets influenced by a lot of things around which can be a simple buttons triggering a service - maybe with an http action - which will set your application in a different state.

## <a name="forRoot">One state on you whole application with forRoot(...)</a>

Lets take an applicationstate for the whole application which is just an object like this

```
{ customerList = [], currentlyLoading = false }
```

The apps application state interface then looks like

{% highlight js %}
interface ApplicationState {
    currentlyLoading: boolean;
    customerList: any []
}

export const initialState: ApplicationState = {
  currentlyLoading: false,
  customerList: []
};
{% endhighlight %}

A reducer could manipulate this state like

{% highlight js %}
export function appReducer(state: ApplicationState = initialState, action: Action) : ApplicationState {
    switch (action.type) {
        case ANYACTION:
            return {
                ...state,
                // modify properties here
            }
        // more actions
        default:
            return state;
    }
}
{% endhighlight %}

which we can use in our whole application like this:

{% highlight js %}
@NgModule({
  imports: [
    BrowserModule,
    StoreModule.forRoot({ applicationState: appReducer})
  ]
})
export class AppModule {}
{% endhighlight %}

So image you have the following angular application structure:

```
app
├── // modules, components and so on
├── app.component.ts
├── app.component.html
├── app.module.ts
└── main.ts
...
```

With the

{% highlight js %}
StoreModule.forRoot({ applicationState: appReducer})
{% endhighlight %}

`forRoot` statement we are applying the state which is the result of the appReducer on the application level. That means that the whole application knows this state and can react to its changes.

The state is now an object with a property called `applicationState` which has the value the appReducers gives us as an output.

But if we console.log out

{% highlight js %}
ngOnInit(): void {
    this.store.pipe(select<any>('applicationState'))
      .subscribe((appState: ApplicationState) => console.log(appState));
  }
{% endhighlight %}

the state we get the following result

`{currentlyLoading: false, customerList: Array(0)}`

So if we log out the initial state we will get back an object with two properties on it. But why is that? Shouldn't it be an object (the one from the `forRoot`) with a property on it called `applicationState`?

Well, the `select` method in the code sample above is literally selecting the property "applicationState" and giving us back the result which is itself an object with two properties `currentlyLoading` and `customerList` on it.

Our state however still is an object like this

```
{
    applicationState: {
            currentlyLoading: false, 
            customerList: Array(0)
        }
}
```

Which we could log out with a statement like

{% highlight js %}
 ngOnInit(): void {
    this.store
      .select<any>((state: any) => state) // the complete state this time!!!
      .subscribe((completeState: any) => console.log(completeState));
  }
{% endhighlight %}

## <a name="forFeature">Separating state into modules with forFeature(...)</a>

So our application state will be influenced my many things we do. We would divide those actions and areas of our application into different modules. Also if you want to keep your application in a structured manner you will separeate it into different modules which provide you more maintainability and overview about what your application is really doing and where you have to search for things like components, maybe pipes, directives etc.

You would possibly introduce a module which encapsulates and abstracts all your customer related things. So you application _could_ look like this:

```
app
└── customers
    ├── components
    ├── containers
    ├── customer.routing.ts
    └── customer.module.ts
├── app.component.ts
├── app.component.html
├── app.module.ts
└── main.ts
...
```

So we created a customers folder where we have a place to put all the customer related stuff into. But now it would be an improvement if only the module itself track of its own state as it does this for its routes and so can fulfill the purpose of a completely seperated part of your application. Also concerning the state. 

So we can apply our module state in a seperate module store which would be a seperate folder in the customers folder:

```
app
└── customers
    ├── components
    ├── containers
    └── store
        └── state/actions/reducers/effects...
    ├── customer.routing.ts
    └── customer.module.ts
├── app.component.ts
├── app.component.html
├── app.module.ts
└── main.ts
...
```

Now we can refactor our application state into the properties which are needed only on application level and not in a feature module because a feature module takes care of its state on its own. Therefore the feature module get its own state.

Ngrx provides a good mechanism for us to apply parts of the state to our main state. We already know the `forRoot()`  method which applies a state on root level. The `forFeature()` method is helping us out merging a state to the root level. So let's try that out: Our AppState is now an empty model (which could have properties however) and the customer store gets the whole properties: 

{% highlight js %}
interface CustomerState {
    currentlyLoading: boolean;
    customerList: any []
}

export const initialState: CustomerState = {
  currentlyLoading: false,
  customerList: []
};
{% endhighlight %}

app.module.ts

{% highlight js %}
@NgModule({
  imports: [
    BrowserModule,
    StoreModule.forRoot({ /* an empty object here for this time */})
  ]
})
export class AppModule {}
{% endhighlight %}


customer.module.ts

{% highlight js %}
@NgModule({
  imports: [
    StoreModule.forFeature('customerFeature', {
      customer: customerReducer
    })
  ],
  exports: [],
  declarations: [],
  providers: []
})
export class CustomerModule {}
{% endhighlight %}

The customer reducer - customer is just a feature module name here - manipulates the whole customerstate now. As our AppState did exactly that before, we can just move the reducer into the customer feature module and rename all the things.

The interesting part is the `forFeature` method above. Lets take a look at this:

{% highlight js %}
StoreModule.forFeature('customerFeature', {
    customer: customerReducer
})
{% endhighlight %}

The `forFeature(...)` method merges an object to the root state which is only an empty object now. It takes the string and creates a property on the root state with that name which has an object itself with the property `customer` which itself again has the two properties from before. So our state would look like this now:

```
{
    customerFeature: {
        customer: {
            currentlyLoading: false, 
            customerList: Array(0)
        }
    }
}
```

If we console log out the _complete_ state this time

{% highlight js %}
 ngOnInit(): void {
    this.store
      .select<any>((state: any) => state) // the complete state this time!!!
      .subscribe((completeState: any) => console.log(completeState));
  }
{% endhighlight %}

we get exactly that result:

```
{
    customerFeature: {
        customer: {
            currentlyLoading: false, 
            customerList: Array(0)
        }
    }
}
```

The `select` method now again gives us the possibility to get a part of the state like this

{% highlight js %}
  ngOnInit(): void {
    this.store
      .select<any>('customerFeature')
      .subscribe((customerState: CustomerState) => console.log(customerState));
  }
{% endhighlight %}

we are selecting the property `customerFeature` and subscribing to that again. So the result of the console log looks like:

```
{
    customer: {
        currentlyLoading: false, 
        customerList: Array(0)
    }
}
```

which would be the same result as we subscribe to the state like

{% highlight js %}
  ngOnInit(): void {
    this.store
      .select<any>((state: any) => state.customerFeature) // no strings here
      .subscribe((customerState: CustomerState) => console.log(customerState));
  }
{% endhighlight %}

## <a name="conclusion">Conclusion</a>

So with the `forFeature(...)` method you can build your state object exactly that modular like you do it with your whole application. Which makes seperation of concerns and not mixing thins very easy and well structured.

Of course all the `any`s can and probably should be replaced with interfaces to be type safe. ;)

You can see examples of states with multiple modules here: [https://github.com/FabianGosebrink/ASPNETCore-Angular-Ngrx](https://github.com/FabianGosebrink/ASPNETCore-Angular-Ngrx)
