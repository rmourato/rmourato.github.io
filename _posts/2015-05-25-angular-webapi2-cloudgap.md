---
layout: post
title: "Cloudgap internals"
description: "A look into how Cloudgap was made"
tags: [AngularJs, C#, WebAPI2]
#comments: true
#share: true
---

About 3 weeks ago (gosh, was it that long ago?) I published [Cloudgap](/2015/05/04/cloudgap-intro.html),
a basic file manager based in AngularJs and Web API 2.

Due to the usual constraints (work, family, etc) I didn't manage to write up an article
on it, so here is my attempt at fixing that.

If you want to skip this and go straight to action, you can find the complete source
code in the [angular-cloudgap](https://github.com/rmourato/angular-cloudgap) GitHub repo.

Interested in knowing how I went about building it? Great!

The project took me about 2 days to do (over 4 Saturday mornings), and a lot of it
was spent Googling around for things like Bootstrap components and getting those to play
along with Angular, so I can safely say it's easy to put together by anyone.

My initial plan was to take you through each individual step, but then I realised this could be
tedious - not that it's a big task, it's not! But the source code is online and you may get faster results
by going straight into it if you are already familiar with the technologies.

With this in mind, I'm going to focus on the following main aspects:

* Cloudgap's goal and it's feature set
* High-level architecture
* Building the UI, without any custom directives
* Designing the services that support the application
* Implementing the backend domain logic for managing the files

# Defining the app's goal

![Cloudgap screenshot](/img/cloudgap/cloudgap_downloads.png)

The app should be very simple to use, but also to implement (both in effort and time).

I want to keep dependencies to a minimum while at the same time not getting
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

# Architecture

![Cloudgap's Architecture](/img/cloudgap/CloudgapArchitecture.png)

The app's architecture is quite simple.

Running on the browser will be the Angular client application. 

The client will speak with the server using the HTTP services exposed by the Web API 2 layer.

These services will accept the requests and delegate them to a manager that will take care of
dealing with file-based operations and returning back to the service what it needs (such as file listings, etc) to respond back to the client.

The files themselves, in this example, will sit outside the application's space as you'll see.

# The UI

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

From these 2 partials **home** will be used as a landing page while **folder** is where all the action happens - file navigation, listing and operations.

Let's jump the the latter which is the one we're interested in.

## Folder partial

If you've seen the screenshot above you may have noticed 3 distinct parts to it:

* File drop area (for uploading files)
* Breadcrumb navigation
* File listing

### File upload

Before we go into detail I'll have to be honest:
I deliberately chose not to write any custom directives for this application.

People tend to jump too quickly into writing them but, really, you should only do
so when you have very good reasons to. They are very flexible, but they can also
increase complexity and will surely lock you to the framework.

As you'll see, the application is simple enough that it doesn't need any. 

Having said this... I did use a 3rd party directive to deal with file upload for me.

While trying to write my own solution I found out file upload can get a bit tricky, especially
with the different behaviours in browsers etc, so I took a shortcut and used Danial Farid's own
[ng-file-upload](https://github.com/danialfarid/ng-file-upload) (thank you! it works great).

Here is how our upload drop area looks like:

{% highlight html %}
<div class="panel panel-default">
    <div ngf-drop ngf-select ng-model="files" ngf-multiple="true"
         ngf-drag-over-class="bg-warning" class="panel-body bg-info">
        Drop files here to upload.
    </div>
</div>
{% endhighlight %}

Nice and simple (because it does hide all the painful detail).

We've configured our directive to allow dropping files, or clicking it to 
present the "File Open" dialog on your browser for selecting 1 or more files to upload.

If you are dragging files over it then the CSS class changes 
to give you feedback that you're dragging files over the control.

To support this, we have the following in our controller:

{% highlight js %}
$scope.$watch('files', function () {
    $scope.upload($scope.files);
});

$scope.upload = function (files) {
    if (files && files.length) {
        for (var i = 0; i < files.length; i++) {
            var file = files[i];
            Upload.upload({
                method: 'POST',
                url: 'api/files',
                data: { FilePath: currentFolder },
                file: file
            }).success(function (data, status, headers, config) {
                $scope.init();
            });
        }
    }
};
{% endhighlight %}

This is pretty much the same as provided by ng-file-upload's example.

The first $scope function allows our directive to be notified for changes
and trigger the upload function. This, in turn, goes through each file in the
list and issues a POST request to our service with the file's contents.

It goes through each file one by one because not all browsers support a single
upload request containing all the files. Instead, we POST each file in turn.

Later on we'll see how we handle these on the server.

### Breadcrumb navigation

{% highlight html %}
<ol class="breadcrumb">
    <li ng-repeat-start="crumb in breadcrumb" ng-if="!$last">
        <a ng-href="#{{crumb.Href}}"><span ng-bind="crumb.Name"></span></a>
    </li>
    <li class="active" ng-repeat-end ng-if="$last">
        <span ng-bind="crumb.Name"></span>
        <button class="btn btn-primary btn-xs" ng-click="createNewFolder()">
            <i class="glyphicon glyphicon-plus"></i> <span ng-bind="createFolder.Name"/>
        </button>
        <input type="text" placeholder="Type a folder name..." 
               ng-show="createFolder.Open" 
               ng-model="createFolder.Input"/>
    </li>
</ol>
{% endhighlight %}

There are a few thing going on here.

The first list item declaration is where we build each bit of our breadcrumb.

**breadcrumb** is an array of objects, each with a Name and Href properties, and
ng-repeat will go through each and generate a list item for it.

The second declaration only takes effect for the last object being iterated through (see
how ng-repeat-end is used).

We don't want the last breadcrumb to generate a link as it'll correspond to the
current folder in view so we don't emit an anchor here.

Let's look at the controller's code that supports this.

{% highlight js %}
var baseUrl = $location.path().split("/")[1];
var currentFolder = $routeParams.folderName;

var appendToUrl = function (url, resource) {
    return url + "/" + resource;
};

var getCrumbs = function () {
    var tokens = currentFolder.split("/");
    var crumbs = [];
    var ref = baseUrl;
    for (var i = 0; i < tokens.length; ++i) {
        var token = tokens[i];
        if (!token)
            continue;
        ref = appendToUrl(ref, token);
        var crumb = { Name: token, Href: ref };
        crumbs.push(crumb);
    }
    return crumbs;
};

$scope.breadcrumb = getCrumbs();
{% endhighlight %}

Rather than letting the server control navigation state, we are going to
use the client app's own via Angular's $routeParams' service. **folderName**
corresponds to the currently "open" folder (remember the route definition).

If you run the application in Visual Studio you'll notice we always navigate to
the 'cloudgap' folder as the base. This is so we can immediately present
a starting folder's contents without having to go about creating that first folder.

We use the $location service to extract the partial's base path so we can build the
relative one for each sub-folder (admittedly this sounds hacky but will do the trick for now).

Out **getCrumbs** function will take care of splitting that URL and creating an
object for each path, which will then be added to the array that Angular will use
to generate the HTML.

If you were paying attention to that HTML snippet you might be wondering what
that button and input where for.

{% highlight js %}
var createFolderObj = { Name: "New Folder", Input: "", Open: false };
$scope.createFolder = {};

$scope.createNewFolder = function () {
    var isOpen = $scope.createFolder.Open;
    if (isOpen)
    {
        var newFolderName = $scope.createFolder.Input;
        if (newFolderName.length == 0)
            return;

        // Create folder
        folderService.newFolder(currentFolder, newFolderName).success(function (data) {
            // Load new folder list
            loadFolders(data);
        }).error(function (status, data) {
            alert("Failed to create folder. Got: " + data + status);
        });
    }
    else
    {
        // Open input for user
        $scope.createFolder.Name = "Create";
        $scope.createFolder.Open = true;
    }
};
{% endhighlight %}

The button is there to we allow a user to create another folder at the same level.
When clicking on it we'll be displaying an input to accept the new folder's name and then 
use it to request the server to create it for us.

We'll use the "Open" property of our object to toggle between the control being active (allowing input)
or not (presenting just the button).

### File Listing

The final but most important part of the UI will be to present those files and folders.

{% highlight html %}
<div class="panel panel-default">
    <table class="table table-hover">
        <tbody>
            <tr ng-repeat="item in folder.Contents | orderBy: ['-Type', 'Name']">
                <td ng-click="navigateToItem(item)">
                    <i class="glyphicon glyphicon-file" 
                       ng-if="isFile(item)"></i>
                    <i class="glyphicon glyphicon-folder-close" 
                       ng-if="!isFile(item)"></i>
                    <span ng-bind="item.Name"/>
                </td>
                <td class="text-right">
                    <button class="btn btn-xs btn-danger" 
                            ng-click="deleteItem(item)" 
                            ng-if="item.FileCount === 0">
                        <i class="glyphicon glyphicon-trash"></i>
                    </button>
                </td>
            </tr>
        </tbody>
    </table>
</div>
{% endhighlight %}

To list the files we build a table, each row having 2 columns.

The 1st column will present an icon for the appropriate type of item, file
or folder, followed by that item's name.

The 2nd column, at the right-most side of your screen, will have a button
allowing the user to  delete that item. 

We will only allow deleting files or empty folders to prevent wiping an
entire branch by mistake.

Let's look at the corresponding controller's code for it:

{% highlight js %}

var loadFolders = function (data) {
    $scope.folder = data;
};

var downloadFile = function (file) {
    var filepath = "api/files?file=" + encodeURIComponent(appendToUrl(currentFolder, file.Name));
    window.location.assign(filepath);
};

$scope.navigateToItem = function (item) {
    if (item.Type === "folder") {
        var newPath = appendToUrl($location.url(), item.Name);
        $location.path(newPath);
    }
    else
        downloadFile(item);
};

$scope.deleteItem = function (item) {
    folderService.deleteItem(currentFolder, item.Name).success(function (data) {
        // Load new folder list
        loadFolders(data);
    }).error(function (status, data) {
        alert("Failed to delete item. Got: " + data + status);
    });
};

$scope.isFile = function (item) {
    return item.Type === "file";
};

$scope.init = function () {
    folderService.listFolder(currentFolder).success(function (data) {
        loadFolders(data);
    }).error(function (status, data) {
        alert("Err. Got: " + data + status);
    });
};

$scope.init();

{% endhighlight %}

This should be fairly straightforward.

When the partial first loads, it hits the server to get the file list
for the current path. Loading it into our page is a matter of setting
the **folder** $scope object to the one returned by the server - nothing
more than a collection of objects with a Name, Type (file/folder) and 
FileCount (only relevant for folders) properties.

Clicking an item will have the controller navigate to it (if a folder), by changing
the app's $location to the new one, or download it (if a file), by using the browser's
window.location.assign function to hit the server and present the user with a "Save As"
dialog on receiving the response.

The "delete" button is a simple call to the server to get rid of the selected item
from the current folder.

# HTTP Services

Great, so we have our UI in place but without a backend to actually give us
things like the file listing or allowing us to upload and download files it's nothing
but an empty shell.

We'll start from the frontend again and see how we can arrange the service layer in Angular,
followed by the server implementation in Web API 2.

## Angular Services

Luckily, our service layer is dead easy and lends itself nicely to REST - 
no wonder, web servers have been servings static files since the beginning of the internet
so it's no surprise that HTTP verbs (actions) and resources (files) adhere to this.

{% highlight js %}
.service('folderService', function ($http) {
    var url = "api/folders";

    this.listFolder = function (path) {
        return $http.get(url, { params: { folderName: path } });
    };

    this.newFolder = function (path, name) {
        var item = { Path: path, Name: name };
        return $http.post(url, item);
    };

    this.deleteItem = function (path, name)
    {
        return $http.delete(url, { params: { path: path, name: name } });
    }
})
{% endhighlight %}

Our API URL is at "/api/folders" and it offers us 3 functions for a given path:

* Listing all files/folders (GET)
* Creating a named folder (POST)
* Deleting a file/folder (DELETE)

The other service that may go unnoticed is the one for uploading and downloading files,
located at "api/files", used when we call window.location.assign (download) or
drag and drop files onto the upload area.

How does the server implement these, then?

## Web API 2 API

We saw earlier that we have two API paths, "folders" and "files", so this means
that the server offers 2 Web API 2 controllers, FoldersController and FilesController
respectively.

### Folders

The public API for this is the following:

{% highlight csharp %}
public class FoldersController : ApiController
{
    private readonly IFileManager _fileManager;

    public FoldersController() 
    {
        _fileManager = new MyDocsFileManager();
    }

    [HttpGet]
    public FolderDto GetFolderByName(string folderName)
    {
        return GetFolderDtoAt(folderName);
    }

    [HttpPost]
    public FolderDto CreateNewFolder([FromBody]NewFolderDto newFolder) 
    {
        _fileManager.CreateFolder(newFolder.Path, newFolder.Name);
        return GetFolderByName(newFolder.Path);
    }

    [HttpDelete]
    public FolderDto DeleteItem(string path, string name) 
    {
        _fileManager.DeleteItem(path, name);
        return GetFolderByName(path);
    }

    private FolderDto GetFolderDtoAt(string folderPath)
    {
        // Project our files and folders onto DTOs
        var files = _fileManager.GetFiles(folderPath).Select(file => new FileDto
        {
            Name = file.Name,
            Type = "file",
            FileCount = 0
        });
        var folders = _fileManager.GetDirectories(folderPath).Select(f => new FileDto
        {
            Name = f.Name,
            Type = "folder",
            FileCount = f.GetFileSystemInfos().Length // Number of items in said folder
        });

        // Return the result
        var filesAndFolders = files.Concat(folders);
        return new FolderDto { Contents = filesAndFolders };
    }
{% endhighlight %}

As expected 3 services are offered, one to return a folder's contents,
another for creating folders and finally another to delete files or folders.

Each of these returns back the contents of the folder queried/modified
for convenience, so the client can render them without having to ask back for it.

The returned types are described as the file and folder DTOs (Data Transfer Objects),
which you will now recognize from the Angular's partial and controller definitions:

{% highlight csharp %}
public class FileDto
{
    public string Name { get; set; }
    public string Type { get; set; }
    public int FileCount { get; set; }
}    

public class FolderDto
{
    public IEnumerable<FileDto> Contents { get; set; }
}
{% endhighlight %}

Finally, an inbound DTO is used for describing the Name and Path for new folders to
be created by the CreateNewFolder service:

{% highlight csharp %}
public class NewFolderDto
{
    public string Path { get; set; }
    public string Name { get; set; }
}
{% endhighlight %}

### Files

Our final controller will deal with the download (GET)
and upload (POST) of files:

{% highlight csharp %}
public class FilesController : ApiController
{
    private readonly IFileManager _fileManager;

    public FilesController() 
    {
        _fileManager = new MyDocsFileManager();
    }

    [HttpGet]
    public HttpResponseMessage GetFile (string file)
    {
        HttpResponseMessage response = 
            new HttpResponseMessage(HttpStatusCode.OK);
        var stream = _fileManager.GetOpenFileStream(file);
        response.Content = new StreamContent(stream);
        response.Content.Headers.ContentType = 
            // Trigger "save as" on client
            new MediaTypeHeaderValue("application/octet-stream");
        response.Content.Headers.ContentDisposition = 
            new ContentDispositionHeaderValue("attachment")
            {
                // Set a default for the client's file name
                FileName = _fileManager.GetFileName(file) 
            };
        return response;
    }

    [HttpPost]
    public async Task<HttpResponseMessage> Upload()
    {
        // Read the file and form data
        var provider = new MultipartFormDataMemoryStreamProvider();
        await Request.Content.ReadAsMultipartAsync(provider);

        // Extract the fields from the form data
        var formData = provider.FormData;
        if (formData.HasKeys())
        {
            var fileDesc = GetFormDataDto<FileDescriptorDto>(formData);
            
            // Check if files are on the request
            if (provider.FileStreams.Count > 0)
            {
                _fileManager.SaveFiles(provider.FileStreams, 
                                       fileDesc.FilePath);
                return Request.CreateResponse(HttpStatusCode.OK);
            }
        }

        return Request.CreateResponse(HttpStatusCode.InternalServerError, 
                                      "Failed to handle file upload");
    }

    private T GetFormDataDto<T>(NameValueCollection formData)
    {
        var unescapedFormData = 
            Uri.UnescapeDataString(formData.GetValues(0).FirstOrDefault());
        return JsonConvert.DeserializeObject<T>(unescapedFormData);
    }
}
{% endhighlight %}

#### Download

A few things are worth noting here.

The GetFile service sets the response's content from a file stream.
You would expect us having to Dispose of it however this will be
dealt with for us.

Setting the Content-Type to "application/octet-stream" and Content-Disposition
to "attachment" will have the client pop up a "Save As" dialog for us on
receiving the response.
The type allows for arbitrary data to be returned so it'll cover for any
type of file being downloaded.

Finally, having the FileName on the attachment set means the browser can use
it as the default file name to use on the save dialog rather than some arbitrary name.

#### Upload

The file data will travel in the HTTP request's body.

Sadly, Web API 2 doesn't offer us a means of extracting this data out into
streams we can use for saving the files.

I have used a modified version of MultipartFormDataMemoryStreamProvider, which was
based on a [StackOverflow](http://stackoverflow.com/questions/19723064/webapi-formdata-upload-to-db-with-extra-parameters/23525195#23525195)'s 
answer to do this for me.

Basically, we will be retrieving the file name and streams of each file uploaded via this provider
and having our file manager implementation store it (more on it in the next section).

# File management

The final piece in our puzzle is to deal with the domain logic, i.e finding, listing,
creating and deleting files and folders.

My file manager has the following contract:

{% highlight csharp %}
interface IFileManager
{
    void CreateFolder(string path, string name);
    void SaveFiles(Dictionary<string, Stream> files, string path);

    FileStream GetOpenFileStream(string filePath);
    string GetFileName(string filePath);
    IEnumerable<DirectoryInfo> GetDirectories(string path);
    IEnumerable<FileInfo> GetFiles(string path);

    void DeleteItem(string path, string name);
}
{% endhighlight %}

The implementer of this interface will decide how each of these actions are done and,
at any time, we can replace one implementation with another should we wish to deal
with files differently (say, storing them in a DB rather than the file system).

To keep things simple I have provided a concrete implementation of this contract
as a file system manager rooted in the current user's "My Documents" folder (this will
be your "/users/username" folder if running under Linux with Mono),

I'll leave the details of it out of this article. File system operations are not that exciting 
and the implementation is very straightforward - do feel free to have a look on the repository.

# Final comments

And that's the gist of it.

Without much effort you can write your own web-based file manager. If 
anything, it should be obvious that such applications are actually not that big a task after all,
though it can certainly get meaty as more features are added to it.

I would certainly *not* use this application in it's current state to host files over the internet.
There is no user authentication and authorisation mechanism in place, no special care
is taken to make sure users don't get access to files outside of it's working path ("/cloudgap") and
it's not packaged in a manner that let's you put it behind a production web server or self-hosted
application, but with a few adjustments you could have it running at home in your local intranet
with relative ease.

I am sure a lot of things could be done more elegantly, improvements made
and lots more features added, but hopefully it was easy to follow, doesn't distract
you with framework ceremonies and let's you get it up and running quickly.
