---
layout: post
title: "Lies, damn lies, and tests"
description: "What is a test for and what makes it a good one"
#modified: 2015-03-01
tags: [TDD, Testing]
image:
  feature: rocket-launch.jpg
  #credit: dargadgetz
  #creditlink: http://www.dargadgetz.com/ios-7-abstract-wallpaper-pack-for-iphone-5-and-ipod-touch-retina/
comments: true
share: true
---

Ahh, the wonderful world of software testing.

For the purpose of this post when I refer to testing I'm talking about automated tests
that build along with the rest of your code and either rely on a framework to run them, such as NUnit or any other
framework relevant to your technology or language, or are custom written
test suites that need to be invoked either manually or via a script.

With that in mind, no one can deny writing tests is a good thing, right?
In theory, with tests in place you are able to catch bugs early in the process, before the customer sees them,
and quickly fix them. They also give you more confidence when refactoring your code because they should be
ensuring the product still works.

Heck, the development community even came up with proven methodologies like Test Driven Development (TDD) and
Behaviour Driven Development (BDD), so everyone must be doing it. And because there
are so many resources out there, be it articles, books, test libraries or full blown frameworks 
they must also be doing it right.

Or is it?

# The purpose of testing

Why do we test our software?

Whenever someone refers to testing in general they usually mean it in terms of
improving the quality of software. In simplistic terms we are ensuring (to some degree) that
 the software written will meet the expectations of the end user, at least in functionality
(it needs to do what I want it to do) and robustness (if it fails then it keeps me from doing what I need to do).

As the software development landscape evolved, though, testing became more than that.

## Behind TDD

When most people are first exposed to TDD their immediate thought is, "I don't have time for this".
The ceremony of creating a new test suite, writing the tests first and continuously changing code to get tests to pass...
Their manager just wants the work done, but naturally wants it bullet proof. What management doesn't want is for the
task at hand to take any longer than it needs to. For you, and because you are already under pressure to deliver, that means no tests.
Let the QA guys do it, they are paid for testing after all.

TDD is not just about testing your software, though. Or a means to make sure tests are written. TDD changes the way your software is designed.
Because in TDD you write your tests first that means you have to think about what needs done first. Questions like
* What is the requirement?
* What entities do I need to model to represent that requirement?
* What will my entities need to do to meet it?

If you don't have a well established requirement do you really know what is being asked of you to do? If not, this
is your immediate opportunity to go and find out. Talk with your Product Manager, Business Analyst -- the customer even, if
you are lucky enough to have that relationship.

After thats in place you can write a small set of tests, expectations of what your software
will need to deliver. Now you think about your entities, their responsibilities, how to name them and 
what they will be asked to do. You declare an instance of their class(es), invoke their methods and assert on them.
None of it will build until you have the skeleton for these so now you go ahead and write
the minimum amount of code required to have your test building, and failing.
It's only at this point that you focus on the business or domain logic needed to get your tests to pass.

You shifted your concern from going straight to the internals, the domain logic, to meeting your requirement.
And along the way the software continuously evolved and gained shape. 

Now that you have tests written you have the added bonus of being able to refactor the internals as much as you need
with extra reassurance that the requirement is still being met, as long as the tests pass.

If the tests are good enough, that is. More on that later.

## Behind BDD

As if TDD wasn't enough of a chore, BDD comes along. Now, you can also mock your entities and give them
new behaviour in order to test some other part of your software.
Once again, BDD introduces more software designing concepts. Because, for mocking dependencies, these 
dependencies need to be abstractions that can be replaced throughout your software. Now you have even
more ceremony to do, writing interfaces and/or abstract classes, coding against abstractions everywhere...

Will the madness ever end? Give me my static class back. I'm sure I can write everything I need inside
a single, 10k lines of code long class, use it everywhere and verify it just as well by actually using the application. 
Proper testing, right?

No. I am only joking.

If you take the time to think you will now realise you are being exposed to SOLID principles, 
such as Dependency Injection (DI). If these
principles are still new to you at this point they will change how you do software design completely. You
will finally understand what loose coupling means and be able to write software in that manner.

That static class you loved? Please bin it now. I do mean it.

# A learning journey

So, applying these methodologies doesn't just let you write tests and catch bugs early. Really,
they teach you to write better software.

I won't lie to you, for someone new to all this you will have a lot to learn. But it's not unless
you give it a genuine effort, and go through the pain of doing so (no, it's not all rainbows and unicorns)
then you will never be able to truly appreciate what's behind all this.
