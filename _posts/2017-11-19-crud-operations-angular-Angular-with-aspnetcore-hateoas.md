---
title: CRUD operations in Angular with ASP.NET Core and HATEOAS
date: 2017-10-20 09:12
author: Fabian Gosebrink
layout: post
tags: aspnetcore angular material hateoas
logo: 'assets/images/logo_small.png'
navigation: true
cover: 'assets/images/aerial-view-of-laptop-and-notebook_bw_osc.jpg'
subclass: 'post tag-speeches'
disqus: true
categories: articles
---

In this blogpost I want to show you how to use CRUD operations in Angular which are driven by an ASP.NET Core WebAPI using HATEOAS to provide several links. 

## First things first

The HATEOAS in this repository does not follow any "standard" like e.g. HAL. But it is enough that you get the idea and an impression how to use it.

I just played around a little bit with this in the last time and maybe you can get some inspiration of how to get stuff going with that maybe in your project. This is only one approach. I would love to hear yours in the comments :-)

## Code

You can find the code here: https://github.com/FabianGosebrink/...

## Overview

1. Take me to [The Backend](#thebackend)
    1. [Customer Controller](#customercontroller)
    2. [The response](#theresponse)
2. Take me to [The Frontend](#thefrontend)
    1. [The data services](#thedataservicesfrontend)
    2. [The components](#thecomponents)
3. Take me to [Links](#links)

## <a name="thebackend">The Backend</a>

The backend is an ASP.NET Core WebAPI which sends out the data as json. With it, every entry contains the specific links and also all links contain the paging links to the next page, previous page etc.

### <a name="customercontroller">Customer Controller</a>

{% highlight js %}


[Route("api/[controller]")]
public class CustomersController : Controller
{
	private readonly ICustomerRepository _customerRepository;
	private readonly IUrlHelper _urlHelper;

	public CustomersController(IUrlHelper urlHelper, ICustomerRepository customerRepository)
	{
		_customerRepository = customerRepository;
		_urlHelper = urlHelper;
	}

	[HttpGet(Name = nameof(GetAll))]
	public IActionResult GetAll([FromQuery] QueryParameters queryParameters)
	{
		List<Customer> allCustomers = _customerRepository.GetAll(queryParameters).ToList();

		var allItemCount = _customerRepository.Count();

		var links = CreateLinksForCollection(queryParameters, allItemCount);

		var toReturn = allCustomers.Select(x => ExpandSingleItem(x));

		return Ok(new
		{
			value = toReturn,
			links = links
		});
	}

	[HttpGet]
	[Route("{id:int}", Name = nameof(GetSingle))]
	public IActionResult GetSingle(int id)
	{
		Customer customer = _customerRepository.GetSingle(id);

		if (customer == null)
		{
			return NotFound();
		}

		return Ok(ExpandSingleItem(customer));
	}

	[HttpPost(Name = nameof(Add))]
	public IActionResult Add([FromBody] CustomerCreateDto customerCreateDto)
	{
		if (customerCreateDto == null)
		{
			return BadRequest();
		}

		if (!ModelState.IsValid)
		{
			return BadRequest(ModelState);
		}

		Customer toAdd = Mapper.Map<Customer>(customerCreateDto);

		toAdd.Created = DateTime.Now;
		_customerRepository.Add(toAdd);

		if (!_customerRepository.Save())
		{
			throw new Exception("Creating an item failed on save.");
		}

		Customer newItem = _customerRepository.GetSingle(toAdd.Id);
		
		return CreatedAtRoute(nameof(GetSingle), new { id = newItem.Id },
			Mapper.Map<CustomerDto>(newItem));
	}

	[HttpDelete]
	[Route("{id:int}", Name = nameof(Delete))]
	public IActionResult Delete(int id)
	{
		Customer customer = _customerRepository.GetSingle(id);

		if (customer == null)
		{
			return NotFound();
		}

		_customerRepository.Delete(id);

		if (!_customerRepository.Save())
		{
			throw new Exception("Deleting an item failed on save.");
		}

		return NoContent();
	}

	[HttpPut]
	[Route("{id:int}", Name = nameof(Update))]
	public IActionResult Update(int id, [FromBody]CustomerUpdateDto updateDto)
	{
		if (updateDto == null)
		{
			return BadRequest();
		}

		var existingCustomer = _customerRepository.GetSingle(id);

		if (existingCustomer == null)
		{
			return NotFound();
		}

		if (!ModelState.IsValid)
		{
			return BadRequest(ModelState);
		}

		Mapper.Map(updateDto, existingCustomer);

		_customerRepository.Update(id, existingCustomer);

		if (!_customerRepository.Save())
		{
			throw new Exception("Updating an item failed on save.");
		}

		return Ok(ExpandSingleItem(existingCustomer));
	}

	private List<LinkDto> CreateLinksForCollection(QueryParameters queryParameters, int totalCount)
	{
		var links = new List<LinkDto>();

		links.Add(
		 new LinkDto(_urlHelper.Link(nameof(Add), null), "create", "POST"));

		// self 
		links.Add(
		 new LinkDto(_urlHelper.Link(nameof(GetAll), new
		 {
			 pagecount = queryParameters.PageCount,
			 page = queryParameters.Page,
			 orderby = queryParameters.OrderBy
		 }), "self", "GET"));

		links.Add(new LinkDto(_urlHelper.Link(nameof(GetAll), new
		{
			pagecount = queryParameters.PageCount,
			page = 1,
			orderby = queryParameters.OrderBy
		}), "first", "GET"));

		links.Add(new LinkDto(_urlHelper.Link(nameof(GetAll), new
		{
			pagecount = queryParameters.PageCount,
			page = queryParameters.GetTotalPages(totalCount),
			orderby = queryParameters.OrderBy
		}), "last", "GET"));

		if (queryParameters.HasNext(totalCount))
		{
			links.Add(new LinkDto(_urlHelper.Link(nameof(GetAll), new
			{
				pagecount = queryParameters.PageCount,
				page = queryParameters.Page + 1,
				orderby = queryParameters.OrderBy
			}), "next", "GET"));
		}

		if (queryParameters.HasPrevious())
		{
			links.Add(new LinkDto(_urlHelper.Link(nameof(GetAll), new
			{
				pagecount = queryParameters.PageCount,
				page = queryParameters.Page - 1,
				orderby = queryParameters.OrderBy
			}), "previous", "GET"));
		}

		return links;
	}

	private dynamic ExpandSingleItem(Customer customer)
	{
		var links = GetLinks(customer.Id);
		CustomerDto item = Mapper.Map<CustomerDto>(customer);

		var resourceToReturn = item.ToDynamic() as IDictionary<string, object>;
		resourceToReturn.Add("links", links);

		return resourceToReturn;
	}

	private IEnumerable<LinkDto> GetLinks(int id)
	{
		var links = new List<LinkDto>();

		links.Add(
		  new LinkDto(_urlHelper.Link(nameof(GetSingle), new { id = id }),
		  "self",
		  "GET"));

		links.Add(
		  new LinkDto(_urlHelper.Link(nameof(Delete), new { id = id }),
		  "delete",
		  "DELETE"));

		links.Add(
		  new LinkDto(_urlHelper.Link(nameof(Add), null),
		  "create",
		  "POST"));

		links.Add(
		   new LinkDto(_urlHelper.Link(nameof(Update), new { id = id }),
		   "update",
		   "PUT"));

		return links;
	}
}

{% endhighlight %}

### <a name="theresponse">The response</a>

So if we now start the WebAPI with `dotnet run` and fire a request to the endpoint `http://localhost:5000/api/customers/` we get the following result

{% highlight json %}

{
    "value": [

        {
            "id": 1,
            "name": "Phil Collins",
            "created": "2017-11-17T20:23:36.2179591+01:00",
            "links": [
                {
                    "href": "http://localhost:5000/api/customers/1",
                    "rel": "self",
                    "method": "GET"
                },
                {
                    "href": "http://localhost:5000/api/customers/1",
                    "rel": "delete",
                    "method": "DELETE"
                },
                {
                    "href": "http://localhost:5000/api/customers",
                    "rel": "create",
                    "method": "POST"
                },
                {
                    "href": "http://localhost:5000/api/customers/1",
                    "rel": "update",
                    "method": "PUT"
                }
            ]
        }
        // ... more of values here
    ],
    "links": [
        {
            "href": "http://localhost:5000/api/customers?pagecount=50&page=1&orderby=Name",
            "rel": "self",
            "method": "GET"
        },
        {
            "href": "http://localhost:5000/api/customers?pagecount=50&page=1&orderby=Name",
            "rel": "first",
            "method": "GET"
        },
        {
            "href": "http://localhost:5000/api/customers?pagecount=50&page=1&orderby=Name",
            "rel": "last",
            "method": "GET"
        }
    ]
}

{% endhighlight %}

Because of the `QueryParameters` we can also fire a requeste like `http://localhost:5000/api/customers?pagecount=10&page=1&orderby=Name` and we can get paging going on over the link there.


## <a name="thefrontend">The Frontend</a>

In the frontend we are using a plain angular cli application and the Angular Material model. I created a small application with three modules

* core
* customer
* app

which you can see in the repository. 

### <a name="thedataservicesfrontend">The data services</a>

The core module is implementing the data services to ensure the communication with the ASP.NET Core WebAPI. 

{% highlight js %}

@Injectable()
export class HttpBaseService {

    private headers = new HttpHeaders();
    private endpoint = `http://localhost:5000/api/customers/`;

    constructor(
        private httpClient: HttpClient) {
        this.headers = this.headers.set('Content-Type', 'application/json');
        this.headers = this.headers.set('Accept', 'application/json');
    }

    getAll<T>() {
        return this.httpClient.get<T>(this.endpoint, { observe: 'response' });
    }

    getSingle<T>(id: number) {
        return this.httpClient.get<T>(`${this.endpoint}${id}`);
    }

    add<T>(toAdd: T) {
        return this.httpClient.post<T>(this.endpoint, toAdd, { headers: this.headers });
    }

    update<T>(url: string, toUpdate: T) {
        return this.httpClient.put<T>(url,
            toUpdate,
            { headers: this.headers });
    }

    delete(url: string) {
        return this.httpClient.delete(url);
    }
}

{% endhighlight %}

The `HttpBaseService` is exposing the http methods towards our application. The interesting part is that the `update` and `delete` methods are getting the complete URL passed as a parameter. We will see where this comes from later. The `add` method is doing a post to the same url as the getall. 

> The url part before the `customer/` can be extracted in a seperate service if you want. This would come from the environment you are running on later.

The spcific `CustomerDataService` then exposes only one method by extending the `HttpBaseService`. It switches around the method type and gets passed as a parameter the choosen method, like doing an Update (PUT), an Add (ADD) or a delete (DELETE).

{% highlight js %}

@Injectable()
export class CustomerDataService extends HttpBaseService {

    fireRequest(customer: Customer, method: string) {

        const links = customer.links.find(x => x.method === method);

        switch (method) {
            case 'DELETE': {
                return super.delete(links.href);
            }
            case 'POST': {
                return super.add<Customer>(customer);
            }
            case 'PUT': {
                return super.update<Customer>(links.href, customer);
            }
            default: {
                console.log(`${links.method} not found!!!`);
                break;
            }
        }
    }
}

{% endhighlight %}

### <a name="thecomponents">The components</a>

If we now want to use this in our component it only has to call the `fireRequest` method with the correct http verb like this:

{% highlight js %}
export class DetailsComponent implements OnInit {

    customer = new Customer();

    constructor(
        private customerDataService: CustomerDataService,
        private route: ActivatedRoute
    ) { }

    ngOnInit() {
        const id = this.route.snapshot.paramMap.get('id');
        this.customerDataService.getSingle<Customer>(+id)
            .subscribe(customer => this.customer = customer);
    }

    save() {

        const method = this.customer.id ? 'PUT' : 'POST';

        this.customerDataService
            .fireRequest(this.customer, method)
            .subscribe((customer: Customer) => this.customer = customer);
    }
}
{% endhighlight %}

Or to get all customers in this case 

{% highlight js %}

export class OverviewComponent implements OnInit {

    dataSource: Customer[];

    constructor(
        private customerDataService: CustomerDataService) { }

    ngOnInit(): void {
        this.getAllCustomers();
    }

    delete(customer: Customer) {
        this.customerDataService.fireRequest(customer, 'DELETE')
            .subscribe(() => {
                this.dataSource =
                    this.dataSource.filter(x => x.id !== customer.id);
            });
    }

    getAllCustomers() {
        this.customerDataService.getAll<Customer[]>()
            .subscribe((result: any) => {
                this.dataSource = result.body.value;
            });
    }
}

{% endhighlight %}

So you can see that for update, add and delete methods only one method with the correct http verb has to be called. The url comes from the entity (customer in this case) itself which gets served over the endpoint coming with its links.

I hope I could give you inspiration of what you can do with Angular and ASP.NET Core. If you made it that far: Thanks for reading.

Fabian


## <a name="links">Links</a>

https://github.com/nbarbettini/BeautifulRestApi