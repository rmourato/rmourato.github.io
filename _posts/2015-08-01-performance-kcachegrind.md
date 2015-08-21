---
layout: post
title: "Performance analysis with KCacheGrind"
description: "Profiling an application with the help of Valgrind, Callgrind and KCacheGrind"
tags: [C, Valgrind, Callgrind, KCacheGrind, Performance, Profiling]
#comments: true
#share: true
---

I'm going to show a simple, yet powerful way to look 
at the performance of an application using Valgrind, Callgrind and
KCacheGrind.

The article is geared towards developers writing software in a Linux environment
and is meant to be a short introduction to the tools.

They are incredibly useful and probably well known by anyone
developing in C and C++, but it's still worth mentioning them here and
showing what they are capable of for quickly analysing a program and 
finding out where optimization opportunities are.

I just had to use them recently and would be lost without them.

If you don't have them installed already, the packages should be readily
available for your favorite distro. In Debian, just install the 
'valgrind' and 'kcachegrind' packages and any dependencies they have
and you should be good to go.

Everything setup up? Let's jump right in then.

# Generating a callgrind analysis

Callgrind is a tool offered as part of Valgrind.

While Valgrind focuses on reporting errors in your application, such as
uninitialized variables, memory leaks and a number of other bugs, callgrind
builds a call graph for your program.

Assuming you have already built your binary, the simplest way of
getting this output is to run the following from your command line,
assuming your terminal is in the same folder as your binary ("app", 
here, is your executable's name):

{% highlight ksh %}
valgrind --tool=callgrind -v ./app
{% endhighlight %}

Now, if you look at the output (callgrind.out.PID) you won't be necessarily delighted, and it
doesn't help us much in terms of looking at performance. This is why we
now feed this into KCacheGrind to give us a graphical representation of
what is going on.

# Looking at KCacheGrind's output

If you only have a single callgrind output in your folder you can simply run:

{% highlight ksh %}
kcachegrind
{% endhighlight %}

Otherwise, specify your run's output by passing it in, e.g:

{% highlight ksh %}
kcachegrind callgrind.out.7719
{% endhighlight %}

Assuming everything went well and it loaded Callgrind's output, you
should now see KCacheGrind's window.

# Profiling a simple app

Take the following program as an example.

{% highlight C %}

#include <stdio.h>
#include <stdint.h>

static int loop (
    const int max)
{
    int i = 0;
    while (i < max)
        ++i;    
    return i;
}

static int recurse (
    const int max,
    int* call_num)
{
    ++(*call_num);
    if (*call_num < max)
        recurse (max, call_num);
    return *call_num;
}

static int find_random (
    const int     max_attempts,
    const uint8_t num_to_find)
{
    const uint8_t max = 255;
    srand (time(NULL));
    
    int hits = 0;
    int i = max_attempts;
    while (i--)
    {
        int rand_num = rand() % max;
        if (rand_num == num_to_find)
            ++hits;
    }
    
    return hits;
}

int main(int argc, char *argv[])
{
    const int max = 100000;
    int call_num = 0;
    
    printf ("Looped %d times\n", loop (max));
    printf ("Recursed %d times\n", recurse (max, &call_num));
    
    uint8_t num_to_find = 55;
    int hits = find_random (max, num_to_find);
    printf ("Found number '%d' %d times over %d attempts\n",
            num_to_find, hits, max);

    return 0;
}

{% endhighlight %}

We have routines that run 100 000 times each. One simply loops through,
the other recurses and another tries counts the number it found '55' 
from a randomly generated number.

The examples are artificial, but the important here is that your application
has to spend most of it's time *somewhere*, and that's what we are trying 
to find out.

So, we build the above, generate a call graph and pass it to KCacheGrind.

You should see something like this:

![KCacheGrind screenshot](/img/kcachegrind/kcachegrind.png)

At the left we can see our program's function calls ordered by total 
time spent, so naturally main() sits pretty much at the top because
that's from where we are calling everything else.
The 'Called' column presents the number of calls for each function, so
we can see each has run 100 00 times as we'd expect.

At the top right corner, with the Callee Map selected, you can see a
block diagram view of the costing of each function relative to others,
clearly showing our find_random (68.91%), recurse (25.52%) and
loop (4.44%) functions as taking the vast majority of the time of our
program, and we can see that find_random has most of it's time spent by the rand
functions.

If you want to focus on a particular function you can simply select it 
and the results shown will be relative to it.

There are a number of other things the tools can give you, 
so do explore!

# "Some of my functions don't show, what gives?"

Well, either it's cost is so insignificant that it sits on the bottom
of the pile (thus skipped), or if your compiler is using optimization
flags then it may have been *inlined*.

Function inlining is an optimization that removes the function call 
altogether and inserts that same code into the caller's, so the parent
function will now be reporting the combined cost of it + your inlined one.

To double-check this is the case, you can run the "nm" utility against
your program and verify that the symbols for those functions are no longer being
exported.

# Appreciate the awesome

Stop for a moment and realise how cool this is - all of this 
insight is provided to us with no effort on our part. We didn't have to change
the application in any way - no code written to timestamp sections of the 
program and report this data out.

Not only is this fantastic while developing, but customers can also use
the same utilities to generate a callgrind output and send it to you 
so you can have a view of your application's behaviour in production,
on the customer's own environment, which may be impossible to reproduce.

I've found these tools to be a life saver, especially when working on a large,
monolithic application. 

Anyone writing code in this kind of environment should have these hand.
