---
layout: post
title: "Running Aurelia on an ASP.NET MVC 5 application"
description: "A walkthrough on how to get Aurelia running from within an ASP.NET MVC 5 application"
tags: [ASP.NET MVC 5, Aurelia, VS 2015, Windows 10]
twitter_img_path: /img/2016/mvc5-aurelia/aurelia-logo.png
#share: true
---

If you have ever worked with Single Page Applications (SPAs) before you 
may have come  across Aurelia, a new framework similar to the likes of
Angular or Ember.

Aurelia struck me for its elegance and simplicity, and I loved how views simply
bind to an ES 2016 JavaScript object (the view-model) with almost no ceremony
or wiring involved.

The Aurelia team has a good set of articles to 
[get you started](http://aurelia.io/docs.html#/aurelia/framework/latest/doc/article/getting-started), but
the skeletons they provide either focus on the SPA alone or demonstrate
how to host it from an ASP.NET vNext / ASP.NET 5 / MVC 6 / ASP.NET Core 1.0
(whatever they call it today) based server application.

While it would be nice to go with the latest and greatest of everything,
if you need to get work done today and want to keep using the ASP.NET stack,
or already have a backend that you don't want to rewrite or migrate, then
ASP.NET MVC 5 is still the stable platform to work on.

I have looked around for steps on how to achieve this but found nothing online,
so I'm filling that gap with today's article.

I am using Visual Studio 2015 on Windows 10 here, but I'm sure you can use
previous versions of either Visual Studio or Windows to the same effect (and
may in fact be easier to setup, as you will soon see).

# Pre-requisites

Gone are the days where you could simply download an archive of JavaScript files,
extract them to your app's folder and link to them from your HTML page to
fire up your SPA.

Nowadays you will find everything resides in separate packages that need
installed separately and parts of them built using a mix of Node.js, npm,
Gulp and others.

Aurelia, and it's dependencies, are offered in this manner.

Before you can do anything, you will need:

* [Node.js](https://nodejs.org/en/)
* [npm](https://github.com/felixrieseberg/npm-windows-upgrade)
* [Git](https://git-scm.com/)

Be sure to follow the links above and read the instructions on how to
install them on Windows. 

This is crucially important, especially for npm because otherwise it will
not work properly. The script will make sure it is upgraded to the latest
version (which we will need) and installed the right way.

After this you may have to restart or log off/on to your user account
to make sure the environment variables are setup properly.

Now, open a console (git bash preferred) and install:

* Gulp

{% highlight console %}
npm install -g gulp
{% endhighlight %}

* jspm

{% highlight console %}
npm install -g jspm
{% endhighlight %}

Up to this point these dependencies are more or less what you'd find
on Aurelia's getting started page, however, because we are on Windows, 
there's additional pain to go through that is best done before we continue further.

And that is...

* [node-gyp](https://github.com/nodejs/node-gyp)

Follow the instructions for your specific OS.

If, like me, you are on Windows 10 64 bit then this is what worked for me
at the time of writing:

* Install the latest [Python v2](https://www.python.org/downloads/) (**not v3**!)
* Make sure [Visual Studio 2015](https://www.visualstudio.com/en-us/products/visual-studio-community-vs.aspx) is installed **with Visual C++ included**
* Install the [Windows 7 64 bit SDK](https://www.microsoft.com/en-us/download/details.aspx?id=8279)

Once again you may need to log off/on again for your environment variable
changes to take effect.

With all this out of our way we can focus on getting that MVC 5 application
hosting an Aurelia SPA.

# Setting up Aurelia on an MVC 5 project

For the next steps I am going to create a new MVC 5 project, but you should
be able to use an existing project of yours to add Aurelia into.

My intent here is to be able to slot Aurelia in without affecting anything
else in the project so you can test it in isolation without side-effects
to the rest of your application.

## Prepare a new MVC 5 solution

Fire up VS 2015 and go to File -> New -> Project...

Choose ASP.NET Web Application targeting your preferred .NET Framework version
(I went with 4.6.1). Now choose the MVC template.

If you are going to do anything with the application then you will mostly
probably want to tick the Web API checkbox, unless you are using another library
to the same effect such as ServiceStack, Nancy or others.

I'm not doing any of that here to keep things simple.

![New MVC5 template](/img/2016/mvc5-aurelia/01_new_project.png)

Now we are going to prepare the folder structure to host the Aurelia SPA.

Navigate to 'Views' and create a new folder named 'Aurelia'.

Right-click it,  create an empty Razor view and name it 'Index'. 

Make sure it's using no layout page - we want our Aurelia app
to run independently of our MVC app, and that includes any styling used in the
project.

![Razor view for the Aurelia app - using no layout](/img/2016/mvc5-aurelia/02_add_view_no_layout.png)

Create a new empty controller for our view. By default, it should create an action
method for you returning our Index view - no changes required.

To finish our MVC app's setup, create a folder named 'Aurelia' in the project's
root directory. This is where we are going to place the Aurelia SPA and all its dependencies.

After these steps, your folder structure should look like this:

![MVC project folder structure for Aurelia](/img/2016/mvc5-aurelia/03_project_folder_structure.png)

## Installing Aurelia

The Aurelia team provides skeletons for a basic app.

Download the [latest skeleton](https://github.com/aurelia/skeleton-navigation/releases/latest) archive
and locate the 'es2016' skeleton - this is the one we'll use.

Copy its contents to our MVC project root 'Aurelia' folder using Windows Explorer.

From VS 2015, toggle the 'Show All Files' button on the Solution Explorer so you can
see these new files. As a minimum you will want to pull in the 'src' and 'styles'
folders, which correspond to the app's HTML, JS and CSS files.

The remaining folders and files are tied to the application's dependencies, build
scripts, configuration and so on which you won't need to touch on just now.

In fact, you don't need to pull anything to your solution at all because
the files will still be served, but it makes sense to have at least the app's 'src'
handy so you can change them and see the effect on the browser.

![Aurelia folder structure](/img/2016/mvc5-aurelia/04_aurelia_folder_structure.png)

Before we can run anything we need to install Aurelia's dependencies.

Open a console on the Aurelia folder and run:

{% highlight console %}
npm install
jspm install -y
{% endhighlight %}

If you followed the Pre-requisites section above entirely then you should
see no errors printed. If you do, please go back and make sure you have
everything needed.

Now test the application by running:

{% highlight console %}
gulp watch
{% endhighlight %}

Open a browser and point it to the URL output by Gulp. If everything
goes well you should see the Aurelia application loaded and running.

## Loading Aurelia from your MVC project

Finally, let's host the application from your MVC project.

Open the Razor view we created for our Aurelia app and change it to the following:

{% highlight html %}
@{
    Layout = null;
}

<!DOCTYPE html>

<html>
<head>
  <meta name="viewport" content="width=device-width" />
  <title>Aurelia</title>
  @Styles.Render("~/Aurelia/styles")
</head>
<body>
  <div aurelia-app="main">
    <div class="splash">
      <div class="message">Aurelia Navigation Skeleton</div>
      <i class="fa fa-spinner fa-spin"></i>
    </div>
    <script src="jspm_packages/system.js"></script>
    <script src="config.js"></script>
    <script>
      System.import('aurelia-bootstrapper');
    </script>
  </div>
</body>
</html>
{% endhighlight %}

This defines a new page that is going load and render the Aurelia app.
To maintain consistency with what you saw when testing the application
with Gulp we are telling the Razor view to include the styles defined
under the name '~/Aurelia/styles'.

This will be defined in our bundle configuration.

Open your BundleConfig.cs file under 'App_Start' and add this to your
RegisterBundles method:

{% highlight csharp %}
bundles.Add(new StyleBundle("~/Aurelia/styles").Include(
            "~/Aurelia/jspm_packages/npm/font-awesome@4.5.0/css/font-awesome.min.css",
            "~/Aurelia/styles/styles.css"));
{% endhighlight %}

Here we are including Aurelia's version of font-awesome and it's styling sheet.

That reference to 'jspm_packages' should have triggered you to think "*Hold on...
that doesn't look right. If I upgrade font-awesome I'll need to manually change
my bundle config again...*".

True. The fact is using ASP.NET to bundle our Aurelia application is probably
not the way to go as Aurelia provides [its own bundling mechanism](http://blog.durandal.io/2015/06/23/bundling-an-aurelia-application/).

The above just makes sure the application has the look and feel you'd expect
when hosted from your MVC project, but this is something you'd have to address
before putting anything in production.

## Running it all

If everything went well you should be able to press F5 on Visual Studio
and point the URL to '/Aurelia/Index'.

This is going to load our Razor view. Any references to the script's in that view,
and Aurelia's dependencies, are done relative to your root '/Aurelia' which is where all 
the application's files are so the application should load, feel and work just the same as when
you ran it using Gulp.

# Ready-made skeleton

If you are feeling lazy you can clone my repository [Mvc5-Aurelia](https://github.com/rmourato/Mvc5-Aurelia)
and follow the instructions.

Because the dependencies are not in source control you will still need to install them,
but at least the MVC side of things has been done for you.

