---
layout: post
title: "C# Closures in Practice - Part 2"
description: "Examples of using closures with C# (Part 2)"
#modified: 2014-10-29
tags: [C#, .NET]
image:
  feature: rocket-launch.jpg
  #credit: dargadgetz
  #creditlink: http://www.dargadgetz.com/ios-7-abstract-wallpaper-pack-for-iphone-5-and-ipod-touch-retina/
comments: true
share: true
---

Part 1 of this article covered the basic constructs for writing a closure
in C# 2.0 and C# 3.0 and above. Part 2 will show some practical examples.

There are many cases where you can use closures, but you should only do so
when it's use is warranted. Remember that it allows access to the parent method's scope,
but you don't need this feature all the time.

A recent example where I've used a closure was on a WinForms application,
to reload a Form after having been closed down.

In my scenario, the Form is used to represent a set of database objects. One
particular object determined the layout and contents to be presented on the Form.
As you may have guessed, it had not been designed to cope with changing this on-the-fly.
Refactoring this would introduce considerable risk as it would represent a complete re-write
and a significant regression testing effort, not to mention that detailed requirements
where nowhere to be seen.
Given this, the safe way to get around the problem is to have the Form shutting down and re-open against
the same set of objects. 

Now, the documented means of determining whether a Form has indeed closed is 
to wire a handler on the FormClosed event, but if you want to re-open it
afterwards then you need which objects the Form is representing so you can reload them.


Now, if we want to deliberately
close the form down and have this handler kicking in we would usually
declare a named method 

The most common examples you will find online will involve event handler
or thread methods declared alongside the respective objects.

