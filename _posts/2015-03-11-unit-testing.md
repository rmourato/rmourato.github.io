---
layout: post
title: "Unit Testing"
description: "A look on how to best size your unit tests."
tags: [Testing]
#comments: true
#share: true
---

How large is the scope of your unit test?

When you write a unit test, do you test a very small unit, or do you go big and try to test as much as you can
with a single test?

And what is a unit, even?

I have seen people writing tests that target a single function/method, a class, a web service...
and calling them all unit tests, so I guess the Unit Under Test (UUT) can be anything.

There is, however, a sweet spot that gives us the most benefits.

# Some benefits of unit testing

To better understand what size each unit should be let's first look at what we can get out from a test.
It can:

* Assist me writing my own code
* Give me reassurance when refactoring it
* Document it
* Find bugs early, ideally immediately after a code build
* Find exactly what's wrong when it fails so we can fix it quickly
* Reduce manual testing effort

And certainly, it should run reliably. 
If it ever fails I want it to be because the code is wrong, not because of setup.

I could keep adding more into the list, but let's stick with these points for now.

# Finding your UUTs

On a talk a had with work colleagues the other day we were discussing where it would be best to
retrofit a test suite for ensuring that a vital piece of functionality wasn't broken, now (they weren't that confident
with the current implementation) or in the future.

The logic lived inside a large set of routines that made up a single transaction, responsible for many other
aspects of the system, and had been subject to change many times due to various issues found over time, so
finally there was some budget put aside to make it more robust.

The system in question has been developed for over 20 years, with a codebase that is largely C and PL/SQL, 
and a whole lot of business logic is kept very tight to the RDBMS, making it a serious challenge for testing.

My proposal was to target the function closest to the logic needing tested. In this manner
the developer could focus on the task at hand, prepare the minimum amount of setup required for the
tests and, if any bugs were found along the way, fix them and update the suite accordingly. 

Another of the developers, though, was arguing that in this manner the test could be passing
whilst the rest of the transaction could still fail. Thus, the test wouldn't prevent
the application from failing should anything with that particular transaction fail.

While he was correct, what this clearly demonstrated is that the transaction should be broken into
more maintainable pieces that can be tested in isolation.

Going with the illusion that by testing more with a single test we are saving ourselves work and making the product more robust 
is **false economy**. What we are doing is hiding design inefficiences that need addressed before they grow even further. 

Let's see why.

If you target a high-level entrypoint for your tests and something wrong happens, your test suite may or may not 
highlight that failure - it all depends on just how many different scenarios you were able to think about.
And even if one of the tests does fail, chances are you will have no clue about *where* exactly it failed because the
test is too distant to the routine you wanted to verify.

It's also likely you will need a lot more setup just to be able to run your tests, as you have to prepare data and whatever
else that transaction depends on. This will make your test more brittle and you will no longer be truly
focusing on your logic but rather be distracted with all it's dependencies.

# Sizing your UUT

The bigger your UUT the less the benefits from it will be.

As tests start failing you'll be sent on a witch hunt. Is it the code that is broke, or is it the test?
Is it because of wrong setup? If not, which library failed, and which routine?

You won't be able to fix your build straight away and probably spend a day or two around it, especially so
if you are not familiar with the area - this defeats a lot of the benefits described above.

And we've probably all heard it similar comments before: 

"*Oh, that test has been failing for a while now. It's because we changed our dataset. Someone needs to fix it sometime.*". 

The team will grow tired of brittle tests and start ignoring them. What you once thought was being tested no longer is, and soon
enough the whole team will be making a mockery of the whole thing and stop writing tests all together.

Don't go there.

If you are writing your code with any object oriented language then keep the test suites focused
on a single class.
Not only is it practical, but it will also tell you things over time and guide 
towards [SOLID](http://en.wikipedia.org/wiki/SOLID_%28object-oriented_design%29) principles.

For instance:

* If your tests grow too much in number then the likelyhood is that your class is being 
responsible for too many things. Follow the Single Responsibilily Principle (SPR) and break it up
into more manageable pieces with a clear intent.
* If your class has too many dependencies and you are finding yourself having to create instances of them
just so you can test your code then it's time to learn about the Liskov Substitution, 
Interface Segregation and Dependency Inversion principles

Learning SOLID will completely change how you think about your code and make testing a breeze.

And if you are writing code with a functional language such as C then, at most, you should focus your test suite
on a single .C code file. You can make the analogy of 1 file per "class". In that manner, you keep all related routines together
inside the same file and the test suite can then focus on that aspect of your application alone and
minimise, or even completely get rid of, any dependencies.

Keep your UUT small and you can have much better control over it.
If you find you are still not getting the test coverage you need then write more test suites 
next to those areas. This way your modules can be reliably tested independently from the rest of your system.

# It's a learning journey. Don't stop.

As you genuinely put an effort in doing testing you will find coding easier,
more enjoyable and testing will be greatly simplified. 

This may be controversial for some, but you also don't need to test *everything*. Sure, that would be nice
and it's certainly what you can achieve by doing pure TDD, but the environment you work on will dictate
just how much effort you can put into testing or not so you need to be pragmatic.
As you become more experienced you will learn to focus that effort on the most valuable business logic and take it
from there as circumstances allow you.

If you follow SOLID then at least your code will be testable by design and enable you to think more in terms of TDD,
but once again that depends on the development process your team uses - that might be an opportunity to teach them too!

Leave good examples behind with your code and you'll be planting the seed for others to learn and extend from.

