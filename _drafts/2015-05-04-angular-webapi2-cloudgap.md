---
layout: post
title: "Cloudgap - Your Own Cloud Service"
description: "An Angular/WebAPI2-based app for managing files over the web"
tags: [AngularJs, C#, WebAPI2]
#comments: true
#share: true
---

Today we are going to build our own cloud service using Angular and Web API 2.

If you want to skip this article and go straight to action, you can find the complete source
code in the [angular-cloudgap](https://github.com/rmourato/angular-cloudgap) GitHub repo.

Interested in knowing what steps to go through into building it? Great!

I must admit I love online file storage services. 

I can just drag and drop stuff onto a web page
and retrieve the files later in pretty much the same manner as using a file explorer.
You can use the service as a backup of your local drive, as a means to copy stuff
from one machine over to another, or even to share files as if it were a network drive.

It is such a simple concept but so handy. 

It does make one wonder why it took so long before these services popped around - well, apart from having the bandwidth and
the storage space, that is.

What I don't appreciate about these, though, is that we are trusting our data
to a 3rd party in *god-knows-where*. As much as I want to believe that my files are safe I can never be too sure about it.

Let's fix that and write our own app for managing our files.

# Defining our goal

![Final product screenshot](/img/2015-05-04-angular-webapi2-cloudgap/cloudgap_downloads.png)

The app should be very simple to use, but also to implement (both in effort and time).

I would like you, as the reader, to be able to quickly go through each step and easily understand
them without losing interest halfway, but due to the technologies chosen you will require at least 
some basic knowledge of AngularJs and .NET (details below).

Because of this I want to keep dependencies to a minimum while at the same time not getting
distracted with time consuming detail that doesn't get us closer to the finished app.

We will be building a [Minimum Viable Product](http://en.wikipedia.org/wiki/Minimum_viable_product)(MVP).
What this means is that our app won't necessarily have all the bells and whistles but 
could be perfecly used as a working product.

## Minimum feature set

File storage services may have a number of features, but as a minimum they should allow us to:

* Upload files (store your stuff)
* Download files (get it back)
* Create folders (organise it)
* Delete files (free up space for other stuff)

With these we can do pretty much everything we need, at the expense of some inefficiencies 
and user experience - for instance, if you wanted to rename a file you'd have to delete it and then re-upload it -
but the fact is you would be able to use such a service.

Writing more features at this point would distract us from our goal - having something that you could quickly implement
and get working.

## Supporting technologies

You could spend days, weeks or even months deciding on the best technology stack to support your solution, but
at the end of the day what you need is a working product.

Either pick the ones you feel most comfortable with or invest in learning technologies you believe
will be worth the investment. If you were working with a team you'd have to take their own capabilities
into account, too.

At this present point in time, and because I feel most comfortable with the .NET stack, 
I believe that choosing [AngularJs](https://angularjs.org/),
a single-page application (SPA) framework to build the frontend, paired with [Bootstrap](http://getbootstrap.com/)
for the presentation and supported by RESTful services written with ASP.NET's [Web API 2](https://msdn.microsoft.com/en-us/library/dn448365%28v=vs.118%29.aspx)
is a good way to approach this problem.

### AngularJs

This client-side framework has had a huge adoption by the community due to it's strongly opinionated, 
but sensible structure to build MV* applications that run in your browser.

Not only has it helped shift the paradigm from developing desktop apps to web-based ones but it also
helped shorten the bridge for building cross-platform applications, capable of running on any device
coupled with an up-to-date browser.

A second incarnation of this framework is currently in the works, Angular 2, that somehow managed to
be almost as controversial as the first, leading to alternatives such as [Aurelia](http://aurelia.io/) to emerge.

### Web API 2

I will have to admit Web API was not my first choice to write the apps' services.

I've been a long time fan of [ServiceStack](https://servicestack.net/) which offers a very clean approach
to writing HTTP-based web services, but since I wanted to offer this solution in it's entirety under the [MIT](http://opensource.org/licenses/MIT) license
it didn't feel right to use ServiceStack this time which was made [AGPL](https://www.gnu.org/licenses/agpl-3.0.html) in it's latest version.

Having said that, I've been wanting to explore Web API for a while and it is similar in many ways to ServiceStack.
This also means we have one less 3rd party dependency to worry about in our app.

## Development tools

The technologies you choose will naturally influence which tools to use.

In this case, get yourself a copy of [Visual Studio Community](https://www.visualstudio.com/products/visual-studio-community-vs) and get cracking.

# Let's get started

## Architecture

![Cloudgap's Architecture](/img/2015-05-04-angular-webapi2-cloudgap/CloudgapArchitecture.png)

The app has a simple architecture.

Running on the browser will be the Angular client application. 

The client will speak with the server using the HTTP services exposed by the Web API 2 layer.

These services will accept the requests and delegate them to a manager that will take care of
dealing with file-based operations and returning back to the service what it needs (such as file listings, etc) to respond back to the client.

The files themselves, in this example, will sit outside the application's space as you'll soon see.

## Client application

Before our application runs we'll have to download the initial payload - the app itself - onto our browser.

This is where we define our dependencies (Bootstrap and Angular) and location for our app's controllers:

{% highlight html %}
<html lang="en">
<head>
    <link rel="stylesheet" href="Vendor/CSS/bootstrap-3.3.4-dist/css/bootstrap.css">
    <script src="Vendor/Scripts/angular-1.3.15/angular.js"></script>
    <script src="Vendor/Scripts/angular-1.3.15/angular-route.js"></script>
    <!-- App -->
    <script src="ngApp/app.js"></script>
    <script src="ngApp/partials/home/home.js"></script>
    <script src="ngApp/partials/folder/folder.js"></script>
</head>
<body ng-app="cloudgap">
    <div ng-view></div>
</body>
</html>
{% endhighlight %}

As seen above, our client app will be named "cloudgap". All partials will be injected into the declared *ng-view*.

Our main module, "app.js", will name and define all of the client application's own dependencies:

{% highlight js %}
angular.module('cloudgap', [
  'ngRoute',

  // cloudgap controllers
  'cloudgap.home',
  'cloudgap.folder'
]);
{% endhighlight %}

We can see a dependency on *ngRoute*, Angular's routing module, and two of our controllers,
*home* and *folder*.

Let's see what these routes look like:

{% highlight js %}
.config(function appConfig($routeProvider)
{
    $routeProvider.
    when('/home/', {
        templateUrl: 'ngApp/partials/home/home.html',
        controller: 'HomeCtrl'
    }).
    when('/folders/:folderName*', {
        templateUrl: 'ngApp/partials/folder/folder.html',
        controller: 'FolderCtrl'
    }).
    otherwise({
        redirectTo: '/home'
    });
})
{% endhighlight %}

Our route definition says:

* Redirect any URL to '/home/', unless it has been specifically set to it or to the '/folders/' URL
* If '/home/' has been reached, replace *ng-view* with the 'home.html' partial and assign it with the 'HomeCtrl' controller
* If '/folders/' has been reached, replace *ng-view* with the 'folder.html' partial and assign it with the 'FolderCtrl' controller
* Let 'folderName' be a route parameter to represent anything after the '/folders/' URL and make it accessible by the assigned controller

From these 2 partials:

* **home**, will be used as a landing page
* **folder**, is where all the action happens - file navigation, listing and operations

Let's jump the the latter which is the relevant to us.

### Folder partial

If you've seen the screenshot above you may notice 3 distinct parts to it:

* File drop area (for uploading files)
* Breadcrumb navigation
* File listing

