---
id: 395
title: Unit of Work and the Repository Pattern with Entity Framework 0.6.1
date: 2014-08-22T16:00:21+00:00
author: Fabian Gosebrink
layout: post
tags: ef entityframework nuget unitofwork 
logo: 'assets/images/logo_small.png'
navigation: True
cover: 'assets/images/download_edit_dark.jpg'
subclass: 'post tag-speeches'
disqus: true
categories: articles
---

Update: Solution did not have the latest Version running. So the example-Solution has been updated! Enjoy.

===========================

I just tried to implement a solution to get the UnitOfWork for the EntityFramework generic (see [this](http://offering.solutions/2014/07/01/asp-net-mvc-architecture-part-iii-generic-repositories-and-unitofwork/ "ASP.NET MVC Architecture (Part III): Generic Repositories and UnitOfWork") post) and put all this into a NuGet-Package. And finally: Here it is!

[UnitOfWork by Offering.Solutions 0.6.1](https://www.nuget.org/packages/OfferingSolutions.UnitOfWork.Structure/0.6.1)

This blogpost has a sample-solution attached. Feel free to look into it. But before you do let me explain this package a bit. It will use the UnitOfWork, you will be able to use GenericRepositories and Customrepositories and everything is cached so that if you use the repositories in one using they only have to be created once.

I wont go into details about the sense of the UnitOfWork-thing but I want to give you code-examples how you can start with this in just a few minutes of your time.

If you want to work with the UnitOfWork in this NuGet-Package you will have two different possibilities:

  1. Use generic repositories
  2. Extend the generic repositories to use your own functions additional to the given CRUD-operations

I want to handle both cases in this post:

Lets start:

First of all you have to install the NuGet-Package.

It will add you the needed dll you can work with.

### 1) Use generic repositories

The common case is that you have a DatabaseContext and DbSets of you entities stored in it. After adding the package you can use the OsUnitOfWorkContext as follows

```
using (IOsUnitOfWorkContext unitOfWorkContext = new OsUnitOfWorkContext(DatabaseContext))
{
	Person person = new Person() { Age = 28, Name = "Fabian" };

	//Adding a new Entity, for example "Person"
	unitOfWorkContext.Add(person);

	//Savechanges
	unitOfWorkContext.Save();

	// Get all Persons
	List<Person> allPersons = unitOfWorkContext.GetAll<Person>();

	// Get all Persons with the age of 35
	List<Person> allPersonsOnAge35 = unitOfWorkContext.GetAll<Person>(x => x.Age == 35);

	// Get all Persons with the age of 35 ordered by Name
	List<Person> allPersonsOnAge35Ordered = unitOfWorkContext.GetAll<Person>(x => x.Age == 35, orderBy: q => q.OrderBy(d => d.Name));

	// Get all Persons with the age of 35 ordered by Name and include its properties
	List<Person> allPersonsOnAge35OrderedAndWithSiblings = unitOfWorkContext.GetAll<Person>(
		x => x.Age == 35,
		orderBy: q => q.OrderBy(d => d.Name),
		includeProperties: "Siblings");

	// Get all Persons and include its properties
	List<Person> allPersonsWithSiblings = unitOfWorkContext.GetAll<Person>(includeProperties: "Siblings");

	// Find a single Person with a specific name
	Person findBy = unitOfWorkContext.FindBy<Person>(x => x.Name == "Fabian");

	// Find a single Person with a specific name and include its siblings
	Person findByWithSiblings = unitOfWorkContext.FindBy<Person>(x => x.Name == "Fabian", includeProperties: "Siblings");

	// Find a person by id 
	unitOfWorkContext.FindSingle<Person>(6);

	//Update an existing person
	unitOfWorkContext.Update(person);

	//Add or Update a Person
	unitOfWorkContext.AddOrUpdate<Person>(person);

	//Deleting a Person by Id or by entity
	unitOfWorkContext.Delete<Person>(6);
	unitOfWorkContext.Delete(person);
}
```

### 2) Use extended repositories

If you want the normal repository to extend a bit with your own functions this is also possible. Everything you have to do is writing your own repository. You can even overwrite the normal CRUD-Methods to do whateer you like in there.

Attention to inherit it from the "IRepositoryContext<YourEntity>" respectively the "RepositoryContextImpl<YourEntity>". The code should look like this:

Interface:

```
public interface IPersonRepository : IRepositoryContext<Person>
{
	void MyNewFunction(int id);
}
```



Implementation:

```
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
```

You can then use it with



```
using (IPersonRepository personRepository = new PersonRepository(new DatabaseContext()))
{
	personRepository.Add(new Person());
	personRepository.Save();
	List<Person> persons = personRepository.GetAll();
	personRepository.MyNewFunction(6);
	//...
}
```

With this you can build your own repositories and they are build up modular and are replacable in an easy way.

Thats it. I think this is nice and smooth. I hope you can get along with it. Have fun.

Regards

Fabian

Download [OfferingSolutions.UnitOfWork.Sample.0.6.1](http://offering.solutions/wp-content/uploads/2014/08/OfferingSolutions.UnitOfWork.Sample.0.6.1.zip)
