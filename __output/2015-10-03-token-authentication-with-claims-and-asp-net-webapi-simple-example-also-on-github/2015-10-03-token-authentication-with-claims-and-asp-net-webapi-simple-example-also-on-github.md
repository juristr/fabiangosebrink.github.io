---
id: 748
title: 'Token Authentication with Claims and Asp.Net WebAPI (Simple Example &#8230; also on GitHub)'
date: 2015-10-03T12:20:52+00:00
author: Fabian Gosebrink
layout: post
tags: asp.net authentication authorization claims oauthprovider roles 
logo: 'assets/images/logo_small.png'
navigation: True
cover: 'assets/images/download_edit_dark.jpg'
subclass: 'post tag-speeches'
disqus: true
categories: articles
---

Hey,

in this post I would like to show you the most simple example about Token Authentication with Asp.Net WebAPI.

The sense behind this is:

  1. We ask the Server for a token
  2. We receive the token, store it client side and&#8230;
  3. &#8230;send it in the header on every request

The "problem" is that we do want to use all build in things Asp.Net WebAPI provides us. Microsoft serves us everything we need. So lets do this 🙂

First of all we configure our WebAPI to create a "controller" which is taking our requests. Here is the first unusual thing: The controller we create is kind of a virtual controller. We only provide it as a string.

<pre class="lang:c# decode:true">OAuthOptions = new OAuthAuthorizationServerOptions
            {
                TokenEndpointPath = new PathString("/Token"),
                Provider = new ApplicationOAuthProvider(),
                AuthorizeEndpointPath = new PathString("/api/Account/ExternalLogin"),
                AccessTokenExpireTimeSpan = TimeSpan.FromDays(14),
                //ONLY FOR DEVELOPING: ALLOW INSECURE HTTP!
                AllowInsecureHttp = true
            };

            // Enable the application to use bearer tokens to authenticate users
            app.UseOAuthBearerTokens(OAuthOptions);</pre>

The "TokenEndpointPath" can be treated like a controller without really having one in your project. You will not find such a class there, so stop looking 😉 Other Properties speak for themselves. Well, now we have to take a look at the ApplicationOAuthProvider, we mentioned in the code, because this is a class which consumes the token request and gives us the token in the end.

Lets have a look at this.

<pre class="lang:c# decode:true ">public class ApplicationOAuthProvider : OAuthAuthorizationServerProvider
    {
        public override async Task ValidateClientAuthentication(OAuthValidateClientAuthenticationContext context)
        {
            context.Validated();
        }

        public override async Task GrantResourceOwnerCredentials(OAuthGrantResourceOwnerCredentialsContext context)
        {
            context.OwinContext.Response.Headers.Add("Access-Control-Allow-Origin", new[] { "*" });

            if(context.UserName != context.Password)
            {
                context.SetError("invalid_grant", "The user name or password is incorrect.");
                return;
            }

            var identity = new ClaimsIdentity(context.Options.AuthenticationType);
            identity.AddClaim(new Claim("sub", context.UserName));
            identity.AddClaim(new Claim(ClaimTypes.Role, "user"));

            context.Validated(identity);

        }
    }</pre>

The first line is a CORS-Line. You can get information about CORS looking [here](http://www.asp.net/web-api/overview/security/enabling-cross-origin-requests-in-web-api) or [here](http://enable-cors.org/server_aspnet.html).

ATTENTION: I am only comparing username and password here for equality. Normally you yould take your own User-Repository or the Asp.Net-Identity thing.

If everything is alright we can create a new identity and add claims to it.

Thats it! For server side.

But how to consume it?

So we have created the enpoint&#8230;lets request it with a POST-Request. (I am using Postman here)![alttext]({{site.baseurl}}assets/articles/2015-10/76233c99-ecbc-4245-8d1f-309334e96284.jpg)So send a post request to the token enpoint we created. Take a look at the "x-www-form-urlencoded" which is very important! Also see the "grant_type" which is set to "password". Without this you will not reach the token endpoint. username and password are equal due to the fact we check it for equality in your OAuthProvider we introduced before.

![alttext]({{site.baseurl}}assets/articles/2015-10/affdf098-b5e7-4c2b-b441-d524f1344c41.jpg)Also Check that in the Headers-Section we set the content-type to "application/x-www-form-encoded". Firing this request reaches the endpoint and is giving us a valid token:

![alttext]({{site.baseurl}}assets/articles/2015-10/ff1f9b2d-f77d-4ed9-abf5-c34c19749537.jpg)There you go. if we now copy this token and send it to a controller we tagged with the [authorize]-Attribute like this:

<pre class="lang:c# decode:true">[Authorize]
    public class ValuesController : ApiController
    {
        // GET api/&lt;controller&gt;
        public IHttpActionResult Get()
        {
            ClaimsIdentity claimsIdentity = User.Identity as ClaimsIdentity;

            var claims = claimsIdentity.Claims.Select(x =&gt; new { type = x.Type, value = x.Value });

            return Ok(claims);
        }
    }</pre>

![alttext]({{site.baseurl}}assets/articles/2015-10/c1a37c15-9d52-4312-bd5d-9ee61810a6d5.jpg)

Note that we added the "Authorization".Header with the "Bearer" and the token we just received. We can send it and receive the protected resource.

Thats it 🙂

You can also check the roles you added in the claims by just mentioning the roles in your Autorize-Attribute:

<pre class="lang:c# decode:true ">[Authorize(Roles = "user")]
    public class ValuesController : ApiController
    {
        // GET api/&lt;controller&gt;
        public IHttpActionResult Get()
        {
            ClaimsIdentity claimsIdentity = User.Identity as ClaimsIdentity;

            var claims = claimsIdentity.Claims.Select(x =&gt; new { type = x.Type, value = x.Value });

            return Ok(claims);
        }
    }</pre>

The roles are added via claims in your OAuthProvider.

Hope this helps anybody.

Happy coding 🙂

[Github repository here](https://github.com/FabianGosebrink/AspNetSimpleWebApiTokenAuthentication)

Fabian
