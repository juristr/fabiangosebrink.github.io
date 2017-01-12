---
title: ASP.NET Unit Of Work with Entity Framework
date: 2015-12-09 21:37
author: Fabian Gosebrink
layout: post
tags: Entity Framework Nuget Unit of Work ASP.NET
logo: 'assets/images/logo_small.png'
navigation: True
cover: 'assets/images/aerial-view-of-laptop-notebook-mobile-phone-and-coffee-cup-on-wooden-table.jpg'
subclass: 'post tag-speeches'
disqus: true
categories: blog articles
---

I just implemented a solution for the ASP.NET Unit Of Work with Entity Framework (see [this](http://offering.solutions/2014/07/01/asp-net-mvc-architecture-part-iii-generic-repositories-and-unitofwork/) post) and put all this into a NuGet-Package. And finally: Here it is!

### Code

Nuget: [UnitOfWork by Offering.Solutions](https://www.nuget.org/packages/OfferingSolutions.UnitOfWork.Structure/)

Github: [OfferingSolutions UnitOfWork with Entity Framework](https://github.com/OfferingSolutions/OfferingSolutions-RepositoryPattern-UnitOfWork)

This blogpost has a sample-solution attached. Feel free to look into it. But before you do let me explain this package a bit. It will use the UnitOfWork, you will be able to use GenericRepositories and Customrepositories and everything is cached so that if you use the repositories in one using they only have to be created once.

I wont go into details about the sense of the UnitOfWork-thing but I want to give you code-examples how you can start with this in just a few minutes of your time.

If you want to work with the UnitOfWork in this NuGet-Package you will have two different possibilities:

1.  Use generic repositories
2.  Extend the generic repositories to use your own functions additional to the given CRUD-operations

I want to handle both cases in this post:

Lets start:

First of all you have to install the NuGet-Package. It will add you the needed dll you can work with.

### 1) Use generic repositories

The common case is that you have a DatabaseContext and DbSets of you entities stored in it. After adding the package you can use the OsUnitOfWorkContext as follows

{% highlight cs %}
using (IOsUnitOfWorkContext unitOfWorkContext = new OsUnitOfWorkContext(new DatabaseContext()))
{
    Person person = new Person() { Age = 28, Name = "Fabian" };

    //Adding a new Entity, for example "Person"
    unitOfWorkContext.Add(person);

    //Savechanges
    unitOfWorkContext.Save();

    //or...
    unitOfWorkContext.SaveASync();

    // Get all Persons
    List allPersons = unitOfWorkContext.GetAll().ToList();

    // Get all Persons with the age of 35
    List allPersonsOnAge35 = unitOfWorkContext.GetAll(x => x.Age == 35).ToList();

    // Get all Persons with the age of 35 ordered by Name
    List allPersonsOnAge35Ordered = unitOfWorkContext.GetAll(x => x.Age == 35, orderBy: q => q.OrderBy(d => d.Name)).ToList();

    // Get all Persons with the age of 35 ordered by Name and include its properties
    List allPersonsOnAge35OrderedAndWithThings = unitOfWorkContext.GetAll(
        x => x.Age == 35,
        orderBy: q => q.OrderBy(d => d.Name),
        includeProperties: "Things").ToList();

    // Get all Persons and include its properties
    List allPersonsWithThings = unitOfWorkContext.GetAll(includeProperties: "Things").ToList();

    // Find a single Person with a specific name
    Person findBy = unitOfWorkContext.GetSingle(x => x.Name == "Fabian");

    // Find a single Person with a specific name and include its siblings
    Person findByWithThings = unitOfWorkContext.GetSingle(x => x.Name == "Fabian", includeProperties: "Things");

    // Find a person by id 
    unitOfWorkContext.GetSingleById(6);

    //Update an existing person
    unitOfWorkContext.Update(person);

    //Add or Update a Person
    unitOfWorkContext.AddOrUpdate(person);

    //Deleting a Person by Id or by entity
    //unitOfWorkContext.Delete(person.Id);
    unitOfWorkContext.Delete(person);
}
{% endhighlight %}

    
### 2) Use extended repositories

If you want the normal repository to extend a bit with your own functions this is also possible. Everything you have to do is writing your own repository. You can even overwrite the normal CRUD-Methods to do whateer you like in there.

Attention to inherit it from the "IRepositoryContext<YourEntity>" respectively the "RepositoryContextImpl<YourEntity>". The code should look like this:

Interface:

{% highlight cs %}
public interface IPersonRepository : IRepositoryContext<Person>
{
    void MyNewFunction(int id);
}
{% endhighlight %}

Implementation:

{% highlight cs %}
public class PersonRepository : RepositoryContextImpl<Person>, IPersonRepository
{
    public PersonRepository(DbContext dbContext)
        : base(dbContext)
    {

    }

    public void MyNewFunction(int id)
    {
        //Do Something
    }
}
{% endhighlight %}

You can then use it with

{% highlight cs %}
using (IPersonRepository personRepository = new PersonRepository(new DatabaseContext()))
{
    personRepository.Add(new Person());
    personRepository.Save();
    List<Person> persons = personRepository.GetAll();
    personRepository.MyNewFunction(6);
    //...
}
{% endhighlight %}

With this you can build your own repositories and they are build up modular and are replacable in an easy way.

Thats it. I think this is nice and smooth. I hope you can get along with it. Have fun.


See <a href="http://www.asp.net/mvc/tutorials/getting-started-with-ef-5-using-mvc-4/implementing-the-repository-and-unit-of-work-patterns-in-an-asp-net-mvc-application" target="_blank">here</a> and <a href="http://codereview.stackexchange.com/questions/31822/unit-of-work-and-repository-design-pattern-implementation" target="_blank">here </a>for inspiration.

See also <a title="ASP.NET MVC Architecture (Part III): Generic Repositories and UnitOfWork" href="http://offering.solutions/2014/07/01/asp-net-mvc-architecture-part-iii-generic-repositories-and-unitofwork/" target="_blank">here </a>for another related BlogEntry (when this Nuget was not released so far ;) )

Regards

Fabian

Download [OfferingSolutions.UnitOfWork.Sample.0.6.1]({{site.baseurl}}assets/articles/wp-content/uploads/2014/08/OfferingSolutions.UnitOfWork.Sample.0.6.1.zip)
