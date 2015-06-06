---
layout: post
title: "Software building blocks vs principles"
description: "Understanding what programming languages, libraries, frameworks and software principles all stand for."
tags: [Software, Principles]
#comments: true
#share: true
---

All too often the concept of frameworks, libraries, languages, patterns... 
are all mixed together, and chances are not many will be paying attention
to what they are and how you reach there - the software principles and concepts behind them.

Say you meet someone at a conference, or a new work colleague at a company lunch.
It's common to ask "so, what are you working on just now"?

Most likely, this is an implicit question for "what are you programming on" and/or
"what technologies are you using".

Recruiters will call you and say "I have a client looking for someone that knows C#, JavaScript,
Angular, CSS...". If you are lucky enough they may also ask you to know "MVC" or some
other acronym for a presentation pattern - ah, but is it about the pattern they are asking you
about or do they just happen to use ASP.NET MVC?

OK, so maybe recruiters are not the best example. After all, they are (probably) not
software engineers themselves but, rather, are tasked with matching the techy words on a CV
with their client's "job spec" and spit out a name and a phone number to call on, but it's
certainly not that less common to have your work colleagues
talking about languages, libraries or frameworks (sometimes interchangeably)
as being the only thing they need to know about to write software.

Before I come across as being pedantic let me explain and clear a bit of the
confusion around all these terms as well.

# Frameworks

"Framework" can be easily used to mean different things but Wikipedia offers a good description:

> A software framework is a universal, reusable software environment that provides particular
functionality as part of a larger software platform to facilitate development of software applications, products and solutions.

So, really, a framework is a collection of ready-made software that developers can use
to build an application on, more productively (or at least that would be the intent). 

Often this collection of software is actually a set of libraries sitting close to each other,
each responsible for one or more tasks that, coming together, provide you 
with the facilities to build complete applications.

Because your application is built on top of the framework, it will define your 
own application's structure.

A good framework will have considered many of the typical problems people find
when building applications to make it easier for you, the developer, to get
past those and get results quicker. Often this is done by hiding those
details away from you so you don't need to worry about them - until you hit an edge case
and then you have to delve into it to figure out what's going on.

A *really* good framework will have abstracted these problems away in such a manner
that minimises the chances of stumbling on these at all and, even if you happen to, provide
a way to cope with them gracefully rather than working around them.

Many applications built on top of frameworks will also have something
that keeps all of it's moving parts together, a runtime, be it because that
application's code needs to be JIT compiled to run on your machine, Java and .NET being popular examples,
or because your code needs interpreted before it runs, such as JavaScript on a browser.

The runtime itself could have been built on top of another framework, and will most
definitely have been built using a miriad of different libraries, coded together
using one or more programming languages, which is one of the reasons 
why the names of frameworks and languages are used interchangeably so often.

In most cases, a developer can easily infer one from the other, for example, if you 
say you code in C# I'm going to guess you are working with the .NET framework.
Likewise, if you say you work with Angular then you'll probably be doing some JavaScript as well.

# Libraries

You could say the building blocks of frameworks are libraries, but this is
not to say that all libraries are necessarily part of frameworks.

A library can be built for the sole purpose of being used
in a framework, yes, but many are more general purpose and allow you to use them
on a wide variety of different applications.

While frameworks are usually tasked to help you build applications, libraries
are tasked of solving one particular class of problems - say, from
low level ones for with memory allocation, to ones providing an access layer
to the file system or a database, or more specific ones for rendering graphics... pretty much any
type of task you can think of will probably already have libraries out there that
deal with them.

What is certain in all of these is that they will have been written using a 
programming language - your way of communicating to the computer what it needs to do.

# Languages

I'm going to quote straight from the wiki again for this one:

> Language is the ability to acquire and use complex systems of **communication**,
particularly the **human** ability to do so, and a language is any specific example of such a **system**.

We all know that around the world people have different cultures and use different languages to communicate between each other,
but in the computer world we also have different CPUs with their own architectures.

Unless you like torture and write pure [machine code](http://en.wikipedia.org/wiki/Machine_code)
then you will be using a programming language to express your instructions to a computer.

Luckily for us, these languages actually abstract us away from the specifics
of the underlying architecure our application is running on.
Even so, we are all well aware there are dozens of programming languages out there. If
the hardware is abstracted away, why don't we just use a single language and stop this
cycle of having to learn a new one every year?

The reason for this is because each language is designed to solve a different set of problems - well,
that and because some languages compete, and/or people don't always agree on what the best
way to solve a problem is.

Still, some are better suited to directly interface with hardware resources, others 
for numeric computation and so on and so forth.

The key here is that a programming language is used to express software constructs
to a computer in a manner that not only *it* can understand, but *we* can as well.

But, what are we trying to describe to the machine after all?

In the real world we talk to people in terms of verbs, such as actions, on subjects,
but a computer would know nothing about skydiving pigs (hint: avoid disapointment by
not searching YouTube for this).

The only thing a computer knows about is to execute some instruction that passes bytes from
one place to another, computes stuff, and stores it back somewhere else.

If the programming languages we used expressed everything in these terms
it would be a nightmare for us to understand it, and that's not great.

Regardless of the language you use, though, the principles you know for
writing software can *always* be used. This, is the stuff that never "gets old"
when a new technology comes around. They are your most valuable asset
in your software engineering career, but also the ones that take the longest
to grasp and apply correctly.

# Software principles and concepts

Principles, concepts, patterns or practices... none of these
are necessarily concerned with the specifics of a given language, 
library or framework, but the reverse is certainly true.

## Basics concepts

Take the example of a pointer - a reference to a memory location.

Regardless of the technology you work with, somewhere in your application,
whether if it's our own code or it's executing runtime, will be logic
for dealing with your array using a pointer for manipulating it in memory,
even if it's a pure CPU register containing the address of the area of
memory you want to access.
Your JavaScript array operations will show none of these details, but ultimately 
they will have to translate to CPU instructions that deal with this.

Now, I'm not saying that everyone needs to understand and be able to follow their code
all the way down to the CPU instructions that it translates to (though it
would give you a tremendous insight about what you are actually coding) but
you certainly should, as a minimum, know the specific concepts that are relevant to the
technologies you do use.

If you code in C or C++ and you don't understand the basic principle behind
a pointer then you should stop writing code and go back to the books, period.
Not only will you not understand how to do things, but you will not understand
them after (if) you've done them.
I have seen, and keep seeing people with *years* of experience
with C/C++, in very respectable positions within their organization,
that still add a few more stars (*) next to their variable when the compiler complains.
You are better off spending the time now to understand why that happens than
all the time spent doing trial and error (and testing) everytime it does.

Each language will have it's own convention for doing things, for example, 
so when you are learning a new one do make sure you know what those conventions are. 
For instance, I keep seeing people writing C# as if it were C++, and put destructors
on every class they write or pass around objects by [ref](https://msdn.microsoft.com/en-GB/library/14akc2c7.aspx)
- it will probably not have the meaning or results you expect. 

Then, for each language you will also have a selection of libraries that
offer common constructs, say lists or hash maps, that you should be familiar
with. Again, always make sure you know how those implementations work 
(any library worth their salt with be well documented)
in order to get reliable results from it.

Once you are comfortable with the rules of your technologies and the
languages they use then you can express broader concepts with them.

## High-level concepts

If you learn a programming language and nothing else you will only be able to
represent instructions to a computer as a set of statements and organize them
(in files or classes).

If you are building anything reasonably sized then you should start bringing
in higher-level concepts to help.

Design patterns (such as Factories or Adapters), design principles 
(such as [SOLID](http://en.wikipedia.org/wiki/SOLID_%28object-oriented_design%29)) --
they will all provide you with a much better insight on how to build software.
Patterns are yet another tool under your belt that will let you identify solutions 
to help you solve a problem better, but design principles such as 
[Dependency Injection](http://en.wikipedia.org/wiki/Dependency_injection)
should be used right from the onset and will allow you to write code in a manner that is 
decoupled, flexible and more easily maintained.

I heard people say before that these approaches are only for people who "don't
write real-world software", but often this comes from individuals who never went 
through the effort of truly understanding them and using them in anger 
in order to truly appreciate the benefits. You will quickly learn after such a 
journey that not using them means a project that is already compromised.

On top of these you can keep applying even higher-level concepts, such as the
likes of [MVC](http://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller)
and [MVVM](http://en.wikipedia.org/wiki/Model_View_ViewModel) presentation patterns, helping you to build applications
that are modular and can have each module tested independently.

Most of these principles can be used regardless of the language or even technology
you use, so knowing them is very valuable.

# Conclusion

If English is your native language and you are trying to learn French, you don't 
expect to be able to mix the two together - the syntax will be different, the
way you build sentences will be different, but the concepts of verbs, subjects and adjectives
will all apply.

A poem will still be a poem in a different language, it will simply be expressed 
in that own language's rules.

In software the same applies, and what matters most is for you to know what those 
concepts are and know them well. Then, all you need to do when you have a new language
in front of you is to learn how to best express those concepts, to communicate
them to your computer and the rest of your team.

You will find that, using this approach, languages, libraries, frameworks - they
are just tools to help you apply the principles you already know to get work
done.

Your exercise then is to find the ones that best fit the problem at hand.
