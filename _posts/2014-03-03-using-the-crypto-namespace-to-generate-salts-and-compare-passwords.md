---
id: 26
title: Using the Crypto-Namespace to Generate Salts and Compare Passwords
date: 2014-03-03T21:02:40+00:00
author: Fabian Gosebrink
layout: post
categories: articles
tags: asp.net crypto hash salt 
logo: 'assets/images/logo_small.png'
navigation: True
cover: 'assets/images/download_edit_dark.jpg'
subclass: 'post tag-speeches'
---

If you are implementing a custom membership- and/or roleprovider (I wont go into custom providers because beside pictures of cats (with or without money or guns) and porn this is one of the most mentioned topics in the web) you should always store passwords with a salt. With the crypted password and the salt you can compare the values during the login-process and set it during the registration process easily by using the Crypto-Namespace of Asp.Net.

```
public bool RegisterUser(string username, string password, bool loginImmediately, string role)
{
    if (UserExists(username))
    {
    throw new MembershipCreateUserException(MembershipCreateStatus.DuplicateUserName);
    }


    User user = new User();
    user.Username = username;
    user.Password = password;
    user.PasswordSalt = Crypto.GenerateSalt();
    user.Password = Crypto.HashPassword(String.Format(&#8220;{0}{1}&#8221;, user.Password, user.PasswordSalt));

    // Add Role and insert User into DB

    if (loginImmediately)
    {
        return WebSecurity.Login(user.Username, password); 
    }

    return registeredSuccessfully;
}
```


The Crypto.GenerateSalt()-Method automatically creates a random salt-value which can be used to hash the password with. This hashed (!) password can now be stored into the database. By using an CustomMembeship-Provider you do not have to miss the WebSecurity-Method the Asp.Net-Framework provides to you.

While verifying the User `VerifyHashedPassword(...)` returns a bool wether the given plain-text password the user gave to you during the login-process salted with the PasswordSalt generated during login compares corectly or not.

```
public bool ValidateUser(string username, string password)
{
    using (DataBaseContext context = new DataBaseContext())
    {
        User user = context.User.FirstOrDefault(x => x.Username == username);
        if (user == null)
        {
            return false;
        }

        return Crypto.VerifyHashedPassword(user.Password, String.Format(&#8220;{0}{1}&#8221;, password, user.PasswordSalt));
    }
}
```

With this code you can store your password safely with saving a lot of work by wusing the .Net-provided Crypto-Namespace.

Fabian