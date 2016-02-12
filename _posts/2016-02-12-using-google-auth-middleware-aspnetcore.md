---
layout: post
title: "Using the Google Authentication Middleware on ASP.NET Core"
description: "Steps for setting up Google Authentication on an ASP.NET Core application"
tags: [ASP.NET Core, MVC 6, vNext, Google, Authentication]
#share: true
---

I had to setup Google Authentication for an application of mine and couldn't find
a definite source to do that. These are the steps I had to follow to do this
for an ASP.NET Core RC1 application using VS 2015.

I assume you will have a Google account to test this with, any will do.

# Create a new project

Start by creating a new "ASP.NET 5" project.

![New ASP.NET 5 project](/img/2016/aspnetcore-gauth/01_new_proj.png)

Once ready, launch it and take note of it's URL on your browser.
Alternatively, check the Project properties and use the URL under Debug ->
Web Server Settings -> App URL.

![Application URL](/img/2016/aspnetcore-gauth/02_app_url.png)

You will use this for the next step.

# Setup a new Google OAuth 2.0 client ID

Open a new browser tab and navigate to the 
[Google Developers Console](https://console.developers.google.com)

You will now create a new Project for your application.

![New Google Dev Project](/img/2016/aspnetcore-gauth/03_g_create_proj.png){: .imgBorder}

Type an appropriate name for it and click Create.

![Google Dev Project Name](/img/2016/aspnetcore-gauth/04_g_proj_name.png){: .imgBorder}

After this, click "Use Google APIs" and enable the "Google+ API".

![Use Google+ API](/img/2016/aspnetcore-gauth/05_g_use_apis.png){: .imgBorder}

Now click "Go to Credentials".

![Goto credentials](/img/2016/aspnetcore-gauth/06_g_goto_credentials.png){: .imgBorder}

Choose "Google+ API" and that you'll be calling it from a Web Server for
accessing User Data.

![Credentials config](/img/2016/aspnetcore-gauth/07_g_web_server_user_credentials.png){: .imgBorder}

Name your client ID and add a redirect URI.

This will be the URL you are using in your ASP.NET project with an additional path,
"signin-google".

Leave the rest blank.

Click "Create client ID".

![OAuth ID creation](/img/2016/aspnetcore-gauth/08_g_oauth_id.png){: .imgBorder}

Now, type in a Product Name and click Continue.

![OAuth consent config](/img/2016/aspnetcore-gauth/09_g_oauth_consent.png){: .imgBorder}

That's it created. Click Done.

Now, open the newly created credentials and take note of the ClientId
and ClientSecret strings.

These will be used soon.

![Google credential details](/img/2016/aspnetcore-gauth/10_g_credentials.png){: .imgBorder}

# Install the Google authentication assembly

Let's go back to our VS project now.

In order to use Google authentication, you first need to install the
"Microsoft.AspNet.Authentication.Google" assembly.

At this point in time there's only a pre-release version. On your
package manager console, type:

{% highlight console %}
Install-Package Microsoft.AspNet.Authentication.Google -Pre
{% endhighlight %}

# Install the Secret Manager

Microsoft provides a command line tool to manage what they call "User Secrets".

These are the sort of things you don't want to check into your source control 
system but want to use whilst in development - connection strings, AWS keys... or,
in this case, our Google OAuth ID and Secret.

Instead of putting this data next to the repository, these go into a file under your
Users folder and ASP.NET Core will fetch them for you when starting up your project - the
AddUserSecrets call you see on our ConfigurationBuilder instance of your Startup class
registers the use of this configuration file.

Before you install this utility, you need to make sure your environment is setup
properly.

Open a command line and type 'dnx'.

If the command isn't recognized, navigate
to where your DXN runtime is installed and update your environment.

Please note that the path below applies to my system at this point in time, but
in your machine please check the runtimes you have installed and 'cd' to the
appropriate one.

{% highlight console %}
cd %userprofile%\.dnx\runtimes\dnx-coreclr-win-x64.1.0.0-rc1-update1\bin
dnvm upgrade
{% endhighlight %}

Hopefully your environment should be OK and now you can install the
user-secret tool. Type:

{% highlight console %}
dnu commands install Microsoft.Extensions.SecretManager
{% endhighlight %}

To give it a try, type 'user-secret'. The command should be recognized and
list the available options.

# Configure your User Secret

From the command line, navigate to the /src folder of your new application -
where your project.json file is located.

Using your Google ClientId and ClientSecret created above, run:

{% highlight console %}
user-secret set Authentication:Google:ClientId <yourId>
user-secret set Authentication:Google:ClientSecret <yourSecret>
{% endhighlight %}

If all went well, you should now have a "secrets.json" file in a folder for your
application under "%APPDATA%\microsoft\UserSecrets\" containing the two keys above.

The remaining step now, then, is configuring the application to use this.

# Enable Google Authentication in your application

Open your Startup.cs.

Just after the Identity middleware, place:

{% highlight csharp %}
app.UseGoogleAuthentication(options=> 
{
	options.ClientId = Configuration["Authentication:Google:ClientId"];
	options.ClientSecret = Configuration["Authentication:Google:ClientSecret"];
});
{% endhighlight %}

Notice how your code is not using the Id and Secret directly - these will
be extracted from your "secrets.json" file and offered to the application
from your Configuration instance.

# Try it out

Launch your application and navigate to the Login page. You will notice
a "Google" button under "Use another service to log in."

Click on it and you should be taken to Google's consent screen:

![Consent screen](/img/2016/aspnetcore-gauth/11_request_for_auth.png){: .imgBorder}

Allow the request - you've now been authenticated in your application
using Google.
