---
layout: post
title: "Languages + libraries + frameworks = software?"
description: "Understanding what programming languages, libraries, frameworks and software principles all stand for, and why they can't realistically live without each other."
tags: [Software, Principles]
#comments: true
#share: true
---

All too often these are mixed together, and chances are not many will be paying attention
to the software principles and concepts behind them.

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
with their client's "job spec" and spit out a name and a phone number to call on.

But it's certainly not that less common to have your work colleagues
talking about languages, libraries or frameworks (sometimes interchangeably)
as if this was the core of what software is.

Before I come across as being pedantic (OK, maybe I am), let me explain.

# Frameworks

I'm starting with the broadest term.

"Framework" can be easily used to mean different things but Wikipedia offers a good description:

> A software framework is a universal, reusable software environment that provides particular
functionality as part of a larger software platform to facilitate development of software applications, products and solutions.

So, really, a framework is a collection of ready-made software that developers can use
to get an application done, quicker. 

Often this collection of software is actually a set of libraries sitting close to each other,
each responsible for one or more tasks that, coming together, provide you 
with the facilities to build complete applications.

What keeps the framework together (it's glue, if any is required), 
and the libraries, are all written in one or more programming languages, which is why
they are mentioned together so often.

In most cases, a developer can easily infer one from the other, for example, if you 
say you code in C# I'm going to guess you are working with the .NET framework.
Likewise, if you say you work with Angular then you'll probably be doing some JavaScript as well.

# Languages

I'm going to quote straight from the wiki again for this one:

> Language is the ability to acquire and use complex systems of **communication**,
particularly the **human** ability to do so, and a language is any specific example of such a **system**.

We all know that around the world people have different cultures and use different languages to communicate between each other,
but in the computer world we also have different CPUs with their own architectures.

Unless you like torture and write pure machine code
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

In the real world we talk to people in terms of verbs (actions) on subjects (objects),
but a computer would know nothing about oranges or skydiving.

The only thing a computer knows about is to execute some instruction that passes bytes from
one place to another, computes stuff, and stores it back somewhere else.

If the programming languages we used expressed everything in these terms
it would be a nightmare for us to understand it 5 minutes after writing that code.

People want their cake and eat it.

# Software principles and concepts

TBD



