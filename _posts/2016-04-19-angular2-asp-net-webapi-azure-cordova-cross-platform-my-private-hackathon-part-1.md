---
id: 906
title: 'Angular2, ASP.NET WebAPI, Azure, Cordova, Cross Platform – My Private Hackathon Part 1'
date: 2016-04-19T17:34:15+00:00
author: Fabian Gosebrink
layout: post
tags: 
logo: 'assets/images/logo_small.png'
navigation: True
cover: 'assets/images/download_edit_dark.jpg'
subclass: 'post tag-speeches'
disqus: true
categories: articles
---

### ![Angular2, ASP.NET WebAPI, Azure, Cordova, Cross Platform – My Private Hackathon]({{site.baseurl}}assets/articles/2016-04-19/a4f664ad-1a00-4812-8a00-c0814964bc7c.jpg)

### Motivation

I had a few free hours and was curious and wanted to know what you can achieve in 9 hours of coding. Due to the fact that I am very into ASP.NET development with Angular I coded a small application which is based on ASP.NET Web API, AngularJs/Angular2 with Typescript & co. So the question is: How far will I go in 9 hours? This is my private Hackathon!!

### Goal : The FoodChooser

When I am not coding I love to stand in the kitchen but I do always have the problem of know knowing what I should cook. I know there are tons of cooking websites out there with a lot of inspiration but my problem is not getting a recipe for the meals but instead I got no idea what to search for. So my idea was to develop a small application where I can manage my food ideas without recipes. Because everybody is doing the meals the way they want to because every taste is different. However: A small application where I can manage my food should be the application I wanted to write: The FoodChooser

### Backlog for first Version

So I wanted to make it basically usable for everyone. So I need a user account or a login for users. The users should add their food items, update and delete them. This is handled the best way in categories or lists. So the users should also create, add, update and delete those lists. The idea is getting random food immediately when getting onto the page without a user account. But every mentioned managing feature should only be available when logged in. But perhaps not every user wants his food items to be public. So users have to switch between publish and unpublishing their food. If you want to get random food only from a list of food items there should also be a button available for this. Because its your food and you don&#8217;t care about public or not: You just want to have a random item out of your food list.

In the end it should also be available for mobile devices. So I need a responsive design which concentrates on the things I need. Basic things. Because I am really not a UI-designer. 😉

To keep a long story short:

  1. Login/Logout
  2. CRUD: Food Items
  3. CRUD: Food Lists
  4. Getting random food from public foods
  5. Getting random food from a single list
  6. Un/Publishing food items
  7. Responsive and easy design also for mobile devices

### Technologies & Tools

The backlog mentioned above are leading to the technologies and tools I wanted to use to get a kick-start. Because ASP.NET 5 is not released yet I used WebAPI 2.2 for this. On client side I wanted to use Typescript with Angular and all tools which are connected to this. This whole thing should be hosted on azure in the end.

  1. ASP.NET WebAPI with Bearer-Token-Identity and OWIN
  2. Typescript
  3. AngularJs and Angular2 Clients
  4. Automapper
  5. Bootstrap
  6. Azure

My tools would be Visual Studio 2015 and Visual Studio Code.

So my timebox was 9 hours. Not a minute more. I know some of the technologies mentioned but not by 100%. Well, lets start coding and learning something!

### The (View)Models

We have CRUD operations for food items, and food lists. And we are going to map them with Automapper. So we need ViewModels for every model.

<pre class="lang:c# decode:true ">public class FoodItem
    {
        [Key]
        public int Id { get; set; }
        public string ItemName { get; set; }
        public FoodList FoodList { get; set; }
        public int FoodListId { get; set; }
        public DateTime Created { get; set; }
        public bool IsPublic { get; set; }
    }</pre>

<pre class="lang:c# decode:true ">public class FoodList
    {
        public int Id { get; set; }
        public string Name { get; set; }
        public string UserId { get; set; }
        public ICollection&lt;FoodItem&gt; Foods { get; set; }
    }</pre>

<pre class="lang:c# decode:true ">public class FoodItemViewModel
    {
        public int Id { get; set; }
        [Required]
        public string ItemName { get; set; }
        public int Rating{ get; set; }
        public int FoodListId { get; set; }
        public DateTime Created { get; set; }
        public bool IsPublic { get; set; }
    }</pre>

<pre class="lang:c# decode:true ">public class FoodListViewModel
    {
        public int Id { get; set; }
        [Required]
        public string Name { get; set; }
        public string UserId { get; set; }
        public ICollection&lt;FoodItem&gt; Foods { get; set; }
    }</pre>

in the OWIN-Configuration we can go ahead and initialize the mappings:

<pre class="lang:c# decode:true">Mapper.Initialize(mapper =&gt;
            {
                mapper.CreateMap&lt;FoodItem, FoodItemViewModel&gt;().ReverseMap();
                mapper.CreateMap&lt;FoodList, FoodListViewModel&gt;().ReverseMap();
            });</pre>

Of course you need to add Automapper to your project which I did via [Nuget](https://www.nuget.org/packages/AutoMapper/4.1.1).

### The API

I mostly start coding server-side.

So the API has to handle FoodItems and FoodLists. Two Models, two controllers.

FoodLists:

[table id=1 /]

Food:

[table id=2 /]

Beside the whole authorization features this is basically it. The API is quite small but I think it clarifies the principle.

<pre class="lang:c# decode:true ">[Authorize]
    [RoutePrefix("api")]
    public class FoodsController : BaseController
    {
        const int MaxPageSize = 10;
        private readonly IFoodRepository _foodRepository;
        private readonly IFoodListRepository _foodListRepository;
        private readonly IRandomNumberGenerator _randomNumberGenerator;

        public FoodsController(IFoodRepository foodRepository, IFoodListRepository foodListRepository,
            IRandomNumberGenerator randomNumberGenerator)
        {
            _foodRepository = foodRepository;
            _foodListRepository = foodListRepository;
            _randomNumberGenerator = randomNumberGenerator;
        }

        [HttpGet]
        [Route("foodlists/{id:int}/foods")]
        public IHttpActionResult GetFoodsFromList(int id)
        {
            try
            {
                FoodList foodList = _foodListRepository.GetSingle(x =&gt; x.Id == id, "Foods");
                return Ok(foodList.Foods.Select(x =&gt; Mapper.Map&lt;FoodItemViewModel&gt;(x)));
            }
            catch (Exception exception)
            {
                return InternalServerError(exception);
            }
        }

        [HttpGet]
        [Route("foodlists/{listId:int}/food/{foodItemId:int}")]
        [Route("foods/{foodItemId:int}", Name = "GetSingleFood")]
        public IHttpActionResult GetSingleFood(int foodItemId, int? listId = null)
        {
            try
            {
                FoodItem foodItem;
                if (listId.HasValue)
                {
                    foodItem = _foodRepository.GetSingle(x =&gt; x.Id == foodItemId && x.FoodList.Id == listId.Value,
                        includeProperties: "FoodList");
                }
                else
                {
                    foodItem = _foodRepository.GetSingle(x =&gt; x.Id == foodItemId, "FoodList");
                }

                if (foodItem == null)
                {
                    return NotFound();
                }

                if (foodItem.FoodList == null || foodItem.FoodList.UserId != CurrentUserId)
                {
                    return StatusCode(HttpStatusCode.Forbidden);
                }

                return Ok(Mapper.Map&lt;FoodItemViewModel&gt;(foodItem));
            }
            catch (Exception exception)
            {
                return InternalServerError(exception);
            }
        }

        [HttpGet]
        [AllowAnonymous]
        [Route("foods/getrandomfood")]
        public IHttpActionResult GetRandomFood()
        {
            try
            {

                IEnumerable&lt;FoodItem&gt; foodItems = _foodRepository.GetAll(x =&gt; x.IsPublic, includeProperties: "FoodList").AsEnumerable();

                if (!foodItems.Any())
                {
                    return NotFound();
                }

                IEnumerable&lt;FoodItem&gt; enumerable = foodItems as IList&lt;FoodItem&gt; ?? foodItems.ToList();
                FoodItem elementAt = enumerable.ElementAt(_randomNumberGenerator.GetRandomNumber(enumerable.Count()));

                if (elementAt == null)
                {
                    return NotFound();
                }

                return Ok(Mapper.Map&lt;FoodItemViewModel&gt;(elementAt));
            }
            catch (Exception exception)
            {
                return InternalServerError(exception);
            }
        }

        [HttpPost]
        [Route("foods")]
        public IHttpActionResult AddFoodToList([FromBody]FoodItemViewModel viewModel)
        {
            try
            {
                if (viewModel == null)
                {
                    return BadRequest();
                }

                if (!ModelState.IsValid)
                {
                    return BadRequest(ModelState);
                }

                FoodList singleFoodList = _foodListRepository.GetSingle(x =&gt; x.Id == viewModel.FoodListId, "Foods");
                FoodItem item = Mapper.Map&lt;FoodItem&gt;(viewModel);
                item.Created = DateTime.Now;
                singleFoodList.Foods.Add(item);
                _foodListRepository.Update(singleFoodList);

                int save = _foodListRepository.Save();

                if (save &gt; 0)
                {
                    return CreatedAtRoute("GetSingleFood", new { foodItemId = item.Id }, Mapper.Map&lt;FoodItemViewModel&gt;(item));
                }

                return BadRequest();
            }
            catch (Exception exception)
            {
                return InternalServerError(exception);
            }
        }

        [HttpPut]
        [Route("foods/{foodItemId:int}")]
        public IHttpActionResult UpdateFoodInList(int foodItemId, [FromBody]FoodItemViewModel viewModel)
        {
            try
            {
                if (viewModel == null)
                {
                    return BadRequest();
                }

                if (!ModelState.IsValid)
                {
                    return BadRequest(ModelState);
                }


                FoodItem singleById = _foodRepository.GetSingleById(foodItemId);

                if (singleById == null)
                {
                    return NotFound();
                }

                singleById.ItemName = viewModel.ItemName;
                singleById.IsPublic = viewModel.IsPublic;

                _foodRepository.Update(singleById);
                int save = _foodRepository.Save();

                if (save &gt; 0)
                {
                    return Ok(Mapper.Map&lt;FoodItemViewModel&gt;(singleById));
                }

                return BadRequest();
            }
            catch (Exception exception)
            {
                return InternalServerError(exception);
            }
        }

        [HttpDelete]
        [Route("foods/{foodItemId:int}")]
        public IHttpActionResult DeleteFoodFromList(int foodItemId)
        {
            try
            {
                FoodItem singleById = _foodRepository.GetSingleById(foodItemId);

                if (singleById == null)
                {
                    return NotFound();
                }

                _foodRepository.Delete(foodItemId);
                int save = _foodRepository.Save();

                if (save &gt; 0)
                {
                    return StatusCode(HttpStatusCode.NoContent);
                }

                return BadRequest();
            }
            catch (Exception exception)
            {
                return InternalServerError(exception);
            }
        }
    }</pre>

FoodListController:

<pre class="lang:c# decode:true ">[Authorize]
    [RoutePrefix("api")]
    public class FoodListsController : BaseController
    {
        private readonly IFoodListRepository _foodListRepository;
        const int MaxPageSize = 10;

        public FoodListsController(IFoodListRepository foodListRepository)
        {
            _foodListRepository = foodListRepository;
        }

        [HttpGet]
        [Route("foodlists")]
        public IHttpActionResult GetAllLists(int page = 1, int pageSize = MaxPageSize)
        {
            try
            {
                if (pageSize &gt; MaxPageSize)
                {
                    pageSize = MaxPageSize;
                }

                IQueryable&lt;FoodList&gt; foodLists = _foodListRepository
                    .GetAll()
                    .Where(x =&gt; x.UserId == CurrentUserId);

                var paginationHeader = new
                {
                    totalCount = foodLists.Count()
                    // Add more headers here if you want...
                    // Link to next and previous page etc.
                    // Also see OData-Options for this
                };

                var result = foodLists
                    .OrderBy(x =&gt; x.Id)
                    .Skip(pageSize * (page - 1))
                    .Take(pageSize)
                    .ToList();

                HttpContext.Current.Response.AppendHeader("X-Pagination", JsonConvert.SerializeObject(paginationHeader));

                return Ok(result.Select(x =&gt; Mapper.Map&lt;FoodListViewModel&gt;(x)));
            }
            catch (Exception exception)
            {
                return InternalServerError(exception);
            }
        }

        [HttpGet]
        [Route("foodlists/{id:int}", Name = "GetSingleList")]
        public IHttpActionResult GetSingleList(int id)
        {
            try
            {
                FoodList singleFoodList = _foodListRepository.GetSingle(x =&gt; x.Id == id);

                if (singleFoodList == null)
                {
                    return NotFound();
                }

                if (singleFoodList.UserId != CurrentUserId)
                {
                    return StatusCode(HttpStatusCode.Forbidden);
                }

                return Ok(Mapper.Map&lt;FoodListViewModel&gt;(singleFoodList));
            }
            catch (Exception exception)
            {
                return InternalServerError(exception);
            }
        }

        [HttpPost]
        [Route("foodlists")]
        public IHttpActionResult AddList([FromBody] FoodListViewModel viewModel)
        {
            try
            {
                if (viewModel == null)
                {
                    return BadRequest();
                }

                if (!ModelState.IsValid)
                {
                    return BadRequest(ModelState);
                }

                FoodList item = Mapper.Map&lt;FoodList&gt;(viewModel);
                item.UserId = CurrentUserId;
                _foodListRepository.Add(item);
                int save = _foodListRepository.Save();

                if (save &gt; 0)
                {
                    return CreatedAtRoute("GetSingleList", new { id = item.Id }, item);
                }

                return BadRequest();
            }
            catch (Exception exception)
            {
                return InternalServerError(exception);
            }
        }

        [HttpDelete]
        [Route("foodlists/{id:int}")]
        public IHttpActionResult DeleteList(int id)
        {
            try
            {
                FoodList singleFoodList = _foodListRepository.GetSingle(x =&gt; x.Id == id, "Foods");

                if (singleFoodList == null)
                {
                    return NotFound();
                }

                if (singleFoodList.UserId != CurrentUserId)
                {
                    return StatusCode(HttpStatusCode.Forbidden);
                }

                _foodListRepository.Delete(singleFoodList);
                int save = _foodListRepository.Save();

                if (save &gt; 0)
                {
                    return StatusCode(HttpStatusCode.NoContent);
                }

                return BadRequest();
            }
            catch (Exception exception)
            {
                return InternalServerError(exception);
            }
        }
    }</pre>

Next we will do the clients

### The Clients

Concerning the clients I was torn between Angular 1.x with Typescript in Visual Studio and Angular 2. Because I could not make a descision whats better I made both. (You can find them in two different approaches in two different GitHub Repositories)

The first approach was the Visual Studio approach with making everything inside VS to be able to make this "right-click &#8211;> Publish"-thing in the end.

The advantage of this is that you can maintain everything from VS like the API, the client code etc. But with this approach you do not have the sexy client- and serverside code separated. We are developing a RESTApi to develop client and server separate. You can handle the client like you want and modify it without touching the server. This is why I tried Angular2 in the second approach making a "client" and "server" folder where the client contains all \*.ts, \*.js and other files and the server contains (the same) REST API we build.

> The GitHub repos are both full functional examples with these different approaches. Choose the one you want. Both do not rely on each other.

So we are building two clients. At this point I could not go for 9 hours anymore, so I build up one client during these 9 hours of coding and build the other one afterwards. Because it&#8217;s fun 🙂

I will describe the clients in other blog posts after this one&#8230;