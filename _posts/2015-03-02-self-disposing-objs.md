---
layout: post
title: "Self-disposing objects."
description: "Why having objects destroying themselves is a bad idea."
tags: [C#, .NET]
image:
  feature: katana.jpg
  credit: seeyouphoto
  creditlink: http://www.flickr.com/photos/seeyou-photos/4122522897
comments: true
share: true
---

The other day I was assigned to look at a bug where the application simply crashed
and no information whatsoever was gathered on why it happened.

This was odd, because said application traps exceptions at the top level and dumps them to a file for later analysis. 
So what could this be?

Well, after following the code through for a bit I found this suspicious System.Windows.Forms derived object
calling Dispose on itself - red flag. From here, it didn't take long to figure out where it was going wrong.

Somewhere on a completely different place of the application a framework routine, 
which keeps track of all forms open by the application, goes through them one by one
and releases allocated resources calling a *native* Win32 routine. 

We have two problems here.

# You created it, you dispose of it

It should always be the responsibility of who creates the resource to free it.

While calling Dispose on an instance multiple times should be fine (as long as the IDisposable pattern has been
implemented property) it should never be the instance itself doing it as it won't know how many objects are referring to it.

If your object commits suicide, any other instances pointing to it can cause your application to fail unexpectedly if they assume they can still use it. 
After all, unless you are tracking it's internal state manually they won't know any better.

# Avoid using OS specific APIs

Unless you have no other way, you should avoid using native routines as they bring with it a number of issues to consider.

If you use them you'll be forfeiting control of your application to outside the framework. Unless done carefully, 
you open yourself for the possibility of these sort of issues occurring and will likely catch other people by surprise.

Your code will also no longer be portable.
Typically people think of .NET applications as something that runs on Windows operating systems only, 
and traditionally it has been the case, but with the existance of the Mono framework and Microsoft open-sourcing
more and more of their frameworks (ASP.NET being the notorious example) that is no longer the case. 

So, before taking the route of going native, have a close look at the .NET APIs to see if there's
a managed alternative to do what you need. Don't get stuck with your knowledge of Win32/MFC if those were the
technologies you worked with the most before.

If you manage to stick with what the .NET framework has to offer, and choose portable 3rd party .NET assemblies
for those extra bits, you will be making your code more maintainable, predictable, and
saving a lot of trouble and head scratching to the other .NET developers in the team.

