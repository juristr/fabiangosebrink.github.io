---
id: 621
title: 'MvcHelpers by Offering.Solutions'
date: 2015-03-04T21:08:01+00:00
author: Fabian Gosebrink
layout: post
tags: mvchelper nuget submitmodel viewmodel 
logo: 'assets/images/logo_small.png'
navigation: True
cover: 'assets/images/download_edit_dark.jpg'
subclass: 'post tag-speeches'
disqus: true
categories: articles
---

Hey,

I just released a new version of the MvcHelpers.

<h3 style="text-align: center;">
  <a href="https://www.nuget.org/packages/OfferingSolutions.MvcHelpers/0.6.0">Nuget: MvcHelpers by Offering.Solutions 0.6.0</a>
</h3>

which includes

  * ViewModelBase
  * SubmitModelBase
  * ModelValidator (for easier validation)

ViewModelBase and SubmitModelBase are two baseclasses for the introduced view-and submitmodels.

<pre class="lang:c# decode:true">public class AccountRegisterViewModel : ViewModelBase
    {
        public AccountRegisterSubmitModel SubmitModel { get; set; }

        public AccountRegisterViewModel()
        {
            Url = "~/Here/Goes/My/Url.cshtml";
        }
    }</pre>

And you can access InfoMessages, SuccessMessages and  ErrorMessages now from your ViewModel.

In your view you are filling up your submitmodel and sending it again to your Server. There you can now validate it:

<pre class="lang:c# decode:true ">public AccountRegisterViewModel RegisterUser(AccountRegisterSubmitModel submitModel)
        {
            AccountRegisterViewModel viewModel = GetAccountRegisterViewMmodel(submitModel);

            try
            {
                if (!_modelValidator.IsSubmitModelValid(viewModel, submitModel))
                {
                    return viewModel;
                }

                // Do some stuff
            }
            catch (Exception)
            {
                viewModel.ErrorMessages.Add("There was an error. Can not register.");
            }

            return viewModel;
        }</pre>

In this exampe the ModelValidator in the NuGet above is used which you can simply register

with Ninject for example like this:

<pre class="lang:c# decode:true">kernel.Bind(typeof(IModelValidator)).To(typeof(ModelValidatorImpl));</pre>

If you validate with the validator, your Errormessages are getting automatically filled with those defined in your submitmodel and you can easily return it. Of course you can do it the long way like this:

<pre class="lang:c# decode:true">ublic AccountRegisterViewModel RegisterUser(AccountRegisterSubmitModel submitModel)
        {
            AccountRegisterViewModel viewModel = GetAccountRegisterViewMmodel(submitModel);

            try
            {
                ReadOnlyCollection&lt;string&gt; errors;
                if (!submitModel.IsValid(out errors))
                {
                    viewModel.ErrorMessages.AddRange(errors);
                    return viewModel;
                }

               // Do some stuff
            }
            catch (Exception)
            {
                viewModel.ErrorMessages.Add("There was an error. Can not register.");
            }

            return viewModel;
        }</pre>

So, this is it. Hope you enjoy!

Regards

Fabian
