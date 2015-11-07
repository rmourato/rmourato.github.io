---
layout: post
title: "How to setup ASP.NET 5 + Visual Studio Code on a Debian VM"
description: "A step-by-step guide on how to setup an ASP.NET 5 development environment to execute and develop applications on the Debian Linux distribution"
tags: [ASP.NET 5, vNext, Visual Studio Code, Debian, Linux, DNX, Yeoman, NodeJs]
#share: true
---

ASP.NET 5 (vNext) is a cross-platform framework for writing web applications -
an unusual step for Microsoft to take but positive for developers out there.

Why? 

Well, if we consider the path Microsoft seems to be taking with Windows 10
(and the web is littered with comments about how that's going) it's good to know
that the .NET ecosystem is slowly detaching itself from what's happening and
it's developers can still continue on working with the framework with less concerns
of it losing traction (though that's another discussion entirely).

Today I'm going through each step you need to go through to prepare a Linux environment
for developing on ASP.NET 5. Better, still! You will be able to take part of
Visual Studio with you so the experience doesn't completely alienate you if
you are not that familiar with Linux in general.

I must say, though. If you are completely new to Linux this post is not going
to teach you anything about the commands you will be running
but will hopefully guide you just enough so you can get things up and running.

For seasoned Linux users, this post is more about consolidating everything that
needs installed so you don't have to go around the internet finding
all the dribs and drabs on the subject. Feel free to skip over steps you already
know about (or even don't entirely agree with!).

Now, get yourself a nice mug of your favourite poison. This 
might take a while.

# Setup a virtual machine

I highly recommend you to prepare a VM to play in, detached from any
machine you may already have and use for development.

The reason for this is that, at this time of writing, the environment is 
still in beta and you may find some hiccups, potentially messing up the
development environment you use to work on.

For this post I will be using Debian, the base Linux distro for many
other flavours of Linux out there, along with the XFCE desktop environment.
The idea is to install the bare minimum you need to have a lightweight
environment where you can try things out.
The version I'm covering here is Debian 8 as that was the latest available
when I was writing this.

So, download [VirtualBox](https://www.virtualbox.org/wiki/Downloads)
and [CD 1 of the latest Debian + XFCE build](http://cdimage.debian.org/cdimage/weekly-builds/amd64/iso-cd/)
(other dependencies will be downloaded during install) for your machine 
(I'm assuming 64bit since that's what most people are running these days).

Downloads complete? Let's get going.

## Setup VirtualBox

Assuming you are on Windows then run the installer you've just downloaded. 

Make sure you don't need your network connection as during the install
it will be temporarily disabled - so finish those downloads you have in
the background first.

Now run VirtualBox and setup a new VM.

1. Click "New"
2. Type a name for your VM (e.g. "Debian DNX")
3. Reserve the appropriate amount of RAM for it (I'd recommend 1GB minimum)
4. Choose "Create a virtual hard disk now"
5. Choose "VDI (VirtualBox Disk Image)"
6. Choose your preferred allocation mode, e.g. "Dynamically allocated" so it only takes as much disk space as needed
7. Now, bump the size of the disk up to some 15GB (in case you play around with the VM for longer than expected)
and choose a location where to store it (click the little folder icon next to the file name)

Before you start that VM, right-click on it from the left-side menu and choose "Settings".

*   General

    Click the Advanced tab and enable bidirectional "Shared Clipboard"
    (this will come in handy).
    
*   System

    Go to the Processor tab and adjust this depending on how many threads
    your CPU has. Why not go for half (e.g. on a 4 thread CPU, choose 2).

*   Storage

    Highlight the CD drive (which should be empty), click the CD icon
    to the right (next to "Optical Drive") and "Choose Virtual Optical Disk Drive".
    Point it to the ISO CD you've just downloaded.

The defaults for the rest should do fine for now.

Select the VM and click "Start".

## Install Debian

You should be presented with a boot menu. Choose "Graphical Install".

From here on follow the instructions, choosing the appropriate keyboard
layout and accepting the defaults for anything you are not familiar with
(such as disk layout, etc).
When choosing software packages, leave only the "XFCE" and "Utilities" 
options ticked.

The setup should download anything else it needs to and prompt you to
remove the install disk before rebooting.
Go to VirtualBox's floating menu -> Devices -> Optical Drives and remove
it. Now let the installer reboot the VM.

If all went well you should then see a login screen. Input your user + 
pass and you will be presented with the XFCE desktop.

Great! That's Debian done. Now, open up a terminal. This will be your
new friend until the end of this post.

For now, install "sudo" by logging in as root:

{% highlight console %}
su -
(enter your password)
apt-get install sudo
{% endhighlight %}

Now, you should add your user to the 'sudoers' list.
There is guidance on how to do this properly online, but since this is a
sandbox VM and you probably want to just get cracking then edit the /etc/sudoers file.
You can use any editor of your liking, the example here uses "nano".

{% highlight console %}
nano /etc/sudoers
{% endhighlight %}

Now copy the entry for 'root' under the section "User privilege specification"
and add another entry like it, replacing "newuser" with your own user name:

{% highlight console %}
# User privilege specification
root    ALL=(ALL:ALL) ALL
newuser ALL=(ALL:ALL) ALL
{% endhighlight %}

Save the file and exit the editor (Ctrl+O, accept changes, then Ctrl+X on nano).

Now leave your "root" session:

{% highlight console %}
exit
whoami
{% endhighlight %}

Make sure "whoami" returns your user name, not "root".

From here on you should not need to run as root any longer. I'll let you
know when there are any exceptions to this rule.

## VirtualBox Guest Additions

You will want to take advantage of your monitor's resolution for your VM,
but first you need to install VirtualBox's guest additions package.

Before you can install it, you will need to install the environment to
build code. This will be required later on as well so this covers it:

{% highlight console %}
sudo apt-get install build-essentials dkms
{% endhighlight %}

Now, on your desktop you may notice a CD icon for these tools. If not, 
go to VirtualBox's floating menu again -> Devices and choose 
"Insert Guest Additions CD image".

When the icon shows on your desktop double-click on it. Take note of the
path where it is and go back to your terminal.

Assuming that path is /media/cdrom (otherwise change it as appropriate):

{% highlight console %}
cd /media/cdrom
sudo sh VBoxLinuxAdditions.run
{% endhighlight %}

The tools should now build and install. When finished, reboot your VM.
When it boots up again your screen should be using the same resolution
as Windows.

# Prepare the .NET execution environment

Now we will be setting up the .NET execution environment, also known as
DNX.

First, though, we will need Mono.

## Installing the Mono runtime

The DNX still depends on [Mono](http://www.mono-project.com/docs/getting-started/install/linux/)
to run. Follow the instructions on that link.

For your convenience, the commands for **Debian 8** are listed here:

{% highlight console %}
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 3FA7E0328081BFF6A14DA29AA6A19B38D3D831EF
echo "deb http://download.mono-project.com/repo/debian wheezy main" | sudo tee /etc/apt/sources.list.d/mono-xamarin.list
sudo apt-get update
echo "deb http://download.mono-project.com/repo/debian wheezy-libjpeg62-compat main" | sudo tee -a /etc/apt/sources.list.d/mono-xamarin.list
sudo apt-get install mono-complete
{% endhighlight %}

After this, you should check as a minimum that you can 
[build and run a simple Mono app](http://www.mono-project.com/docs/getting-started/mono-basics/#console-hello-world).

## Installing DNVM, DNX and libuv

Microsoft provides a [pretty good tutorial](https://docs.asp.net/en/latest/getting-started/installing-on-linux.html#installing-on-debian-ubuntu-and-derivatives)
on how to setup the environment on Debian.

As I've found out, it's best to install DNX both for .NET Core and Mono.

The .NET Core didn't play well with me at this point so we will be using DXN with Mono.

So, for your convenience:

* Install DNVM

{% highlight console %}
sudo apt-get install unzip curl
curl -sSL https://raw.githubusercontent.com/aspnet/Home/dev/dnvminstall.sh | DNX_BRANCH=dev sh && source ~/.dnx/dnvm/dnvm.sh
{% endhighlight %}

* Install DNX for .NET Core

{% highlight console %}
sudo apt-get install libunwind8 gettext libssl-dev libcurl3-dev zlib1g libicu-dev
dnvm upgrade -r coreclr
{% endhighlight %}

* Set it up for Mono

{% highlight console %}
dnvm upgrade -r mono
{% endhighlight %}

* Install libuv

{% highlight console %}
sudo apt-get install make automake libtool curl
curl -sSL https://github.com/libuv/libuv/archive/v1.4.2.tar.gz | sudo tar zxfv - -C /usr/local/src
cd /usr/local/src/libuv-1.4.2
sudo sh autogen.sh
sudo ./configure
sudo make
sudo make install
sudo rm -rf /usr/local/src/libuv-1.4.2 && cd ~/
sudo ldconfig
{% endhighlight %}

# Install the development environment

With the execution environment setup we want something extra to help
us to start writing code. This means installing Yeoman, a scaffolding tool,
and Visual Studio Code, a code editor.

Yeoman depends on NodeJs so we will be installing that first.

## Installing NodeJs

[Follow the instructions](https://nodejs.org/en/download/package-manager/#debian-and-ubuntu-based-linux-distributions)
to install NodeJs for Debian.

Again the commands are listed here. 
It is recommended to do this as root, so:

{% highlight console %}
su -
(enter password)
curl -sL https://deb.nodesource.com/setup_4.x | bash -
apt-get install -y nodejs
{% endhighlight %}

Now, take the chance to update the Node Package Manager (npm)
before you leave your session:

{% highlight console %}
npm install -g npm
exit
{% endhighlight %}

## Installing Yeoman

With NodeJs installed, we can now [install Yeoman](http://yeoman.io/learning/index.html).

{% highlight console %}
sudo npm install -g yo bower grunt-cli gulp
{% endhighlight %}

Now, install the OmniSharp ASP.NET generators:

{% highlight console %}
sudo npm install -g generator-aspnet
{% endhighlight %}

## Installing Visual Studio Code

Go to the [Visual Studio Code](https://code.visualstudio.com/) website
and download the package for Linux.

Code isn't really installed. Rather, you extract it somewhere and run it
from there.

Using the terminal again, unzip the package to a suitable location
and run the Code executable. For example:

{% highlight console %}
mkdir ~/IDEs
unzip VSCode-linux64.zip -d ~/IDEs/
cd ~/IDEs/VSCode-linux-64
./Code &
{% endhighlight %}

If everything went well then Visual Studio Code should start up.

# Your first ASP.NET 5 application

Almost there!

To help you create that first project, Yeoman and the generators we
installed can help.

## Generate a new project

Prepare a folder where you can place a new project. For instance:

{% highlight console %}
mkdir ~/Development/AspNet5-Hello
cd ~/Development/AspNet5-Hello
{% endhighlight %}

Inside it, create a new file named "global.json" and add the following
to it (**note**: the version listed here was true at this point in time
but will likely change again in the near future):

{% highlight json %}
{
  "sdk": {
    "version": "1.0.0-beta8"
  }
}
{% endhighlight %}

Now, from that same folder run the command:

{% highlight console %}
yo aspnet
{% endhighlight %}

Choose "Empty Application" and accept the defaults.

Now navigate to the new app's folder and restore any dependencies the
project has:

{% highlight console %}
dnu restore
{% endhighlight %}

You can now open it.

From Code, go to File -> Open Folder and navigate to "AspNet5-Hello" 
(or whichever name you chose for it).

The folder structure and files should be listed in Code. You can now
use it to edit your application's source and configuration files.

## Run the application

As I understand, a few things don't work yet on Linux. Unfortunately,
one of those thing is debugging ASP.NET 5 apps straight from Code.

For now, run the following from the terminal from your application's
folder (where project.json is located):

{% highlight console %}
dnx web
{% endhighlight %}

If everything starts up OK it should print a URL you can use to connect to.
Copy the URL and point your browser of choice to it.

You are now running your first ASP.NET 5 web app on Linux!

# Last comments

Well, I must admit that was a bit of a ride.

Typical .NET developers are used to fire up Visual Studio and have it
generate a new project for them, clicking Run and that's them sorted.

As we move to Linux, though, we are not quite there yet. It doesn't
help that Linux has so many derivatives either, but would you have ever imagined
doing this a few years back?

Sure, we have had Mono for a while now which already let you setup ASP.NET MVC apps 
(even if you had some setup to do) but having Microsoft pushing towards running
ASP.NET on anything *other* than Windows is uncanny - but clever.

With this they can convince more people to use Azure to run their apps, 
which means more business to them. For developers, though, it represents more choice 
at the expense of more setup/config and a degree of uncertainty
(is this production ready?).

These are certainly not the release-ready versions of the tools and there
is more work to be done, but the landscape is looking promising.
