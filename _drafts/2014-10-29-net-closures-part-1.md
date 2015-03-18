---
layout: post
title: "C# Closures in Practice - Part 1"
description: "Examples of using closures with C# (Part 1)"
#modified: 2014-10-29
tags: [C#, .NET]
image:
  feature: rocket-launch.jpg
  #credit: dargadgetz
  #creditlink: http://www.dargadgetz.com/ios-7-abstract-wallpaper-pack-for-iphone-5-and-ipod-touch-retina/
comments: true
share: true
---

There are a number of articles online about closures, explaining the concept, 
but practical day-to-day examples are not seen that much.

My focus will be to cover the basics and provide a few examples of
where they can be used. I will also demonstrate a few ways of writing them 
depending on the version of the framework you are pegged to because, well,
business constraints may mean that you can't use the latest version of the language.

# The Basics

The use of closures in JavaScript is very common. 
When using frameworks such as AngularJs, for instance, you will be using them all the time - probably 
without giving it too much thought or even realising. It's simply the way to do things.

But, what is a closure, then?

A closure is a function that has access to the environment where it was declared, 
meaning it can use the same variables as it's parent scope. Here's a small JavaScript snippet:

{% highlight js %}
var crap = null;
{% endhighlight %} 

In this example, the child function accesses and presents the value of the
variable that was declared in the parent function's scope. That's it, really.

## C# 2.0 (.NET 2.0)

The earliest support in C# for writing these constructs came in C# 2.0 with
the introduction of anonymous methods, allowing you to write *inline* code such as this:

{% highlight c# %}
var crap = null;
{% endhighlight %} 

This is convenient and enables you to express your logic more concisely,
your method construct is smaller and you are not poluting your class' 
namespace. Naturally, if the same logic where to be used somewhere else too then you 
would write it as a named method instead to keep your code DRY.

As a closure your inner method will be able to access
the same variables as it's parent, meaning you don't need to write methods 
with long signatures and pass arguments around. Everything is right there. 
This leads to good readability.

Anyone, including yourself in a few months time, can go back to your code
and read the entirety of your method's logic without navigating around
the file (or files) to understand it's intent and purpose. You are composing
your method in a manner that keeps all related code together. Neat!

The following version of the language allows you to write this in an even
more compact manner, as you can see next =>

## C# 3.0 (.NET 3.5)

Version 3.0 of the language supersedes the use of anonymous methods in
favour of lambda expressions which, in simplistic terms, is a more convenient 
and terse manner of achieving the same. 
Have a look at the following re-write of the previous example, using a lambda expression instead:

{% highlight c# %}
var crap = null;
{% endhighlight %} 

We are doing the same as before, but with no need for the 'delegate' keyword
or even specifying the type of the arguments - the compiler will infer them
from the target's type.

So now that we know the basic constructs for writing a closure we can look
at practical examples of where to use them.
Stick around for Part 2 to find out!


