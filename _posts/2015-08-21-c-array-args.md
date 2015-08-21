---
layout: post
title: "Typedef'd arrays in C"
description: "Behaviour and use of typedef'd arrays in C"
tags: [C]
#comments: true
#share: true
---

This subject has been a source of confusion where I currently work, with
even very senior devs having the wrong assumption about what's going on,
so I'll try making an attempt at demystifying part of this.

Say you have a definition like this:

{% highlight C %}
typedef char entity_id[32];
{% endhighlight %}

So now we can refer to entity_id *as if it were a new type* (which, as we'll
see, can be misleading).
This may lead us to write something like:

{% highlight C %}
void fill_argument (entity_id arg)
{
    strncpy (arg, "modified", sizeof arg);
}
{% endhighlight %}

Well, at first glance I'd expect my function to fill a copy of 'arg' with
the string "modified" because, after all, I'm not passing 'arg' by pointer, 
so it must be a *copy* of my entity_id, right?

What does this function actually do, and does it behave as intended?
Let's put everything together in a small application and run it to find out.

{% highlight C %}
#include <stdio.h>
#include <string.h>

typedef char entity_id[32];
 
void fill_argument (entity_id arg)
{
    strncpy (arg, "modified", sizeof arg);
}

int main (int argc, char* argv[])
{
    entity_id arg = {'\0'};
    fill_argument (arg);
    printf ("arg has: %s\n", arg);
    return 0;
}
{% endhighlight %}

Build this and run it. You may find it prints:

{% highlight console %}
arg has: modified
{% endhighlight %}

What's going on?

# Arrays are not passed by copy

The 'typedef' may have tricked you, and it most certainly would if you
didn't know it's definition and it wasn't readily available for you (which
tends to be the case on a big codebase).
It makes 'entity_id' look like a structure type, rather than what it really
is - an *alias* for an array of characters. 

You may also not realise the fact that there's no such thing as passing arrays in C - 
doing this will decay the array to a pointer to it's first position, so
the function signature will end up being:

{% highlight C %}
void fill_argument (char* arg)
{% endhighlight %}

And this would build and run just as happily as before. It would even print
exactly the same as before, which may surprise you again. 

It would stop working as soon as you increased the size of the string you are
copying to it:

{% highlight C %}
void fill_argument (char* arg)
{
    strncpy (arg, "modified a bit more", sizeof arg);
}
{% endhighlight %}

Which would print...

{% highlight console %}
arg has: modified
{% endhighlight %}

Oops, we lost the rest of our string.

You will experience this behaviour if you are running this on a machine
with a 64bit architecture, where the size of a pointer are 8 bytes -
which was just enough for copying our first string.

So, how do I write to my 'arg' making sure I don't overrun it?

# Safely writing to typedef'd arrays

Any C dev would quickly say "you need to pass another argument with the size
of your buffer":

{% highlight C %}
void fill_argument_safely (entity_id arg, size_t arg_len)
{
    printf ("arg_len=%zu\n", arg_len);
    strncpy (arg, "modified a bit more", arg_len);
}

int main (int argc, char* argv[])
{
    entity_id arg = {'\0'};
    fill_argument_safely (arg, sizeof arg);
    printf ("arg has: %s\n", arg);
    return 0;
}
{% endhighlight %}

Result:

{% highlight console %}
arg_len=32
arg has: modified a bit more
{% endhighlight %}

That worked fine. But frankly, that doesn't get me terribly excited about it.

After all my 'entity_id' was defined with a size, can I not take advantage of that
so I don't have to grow my function to take another argument?

Well, yes you can:

{% highlight C %}
void fill_argument_safely_and_elegantly (entity_id* arg)
{
    size_t len = sizeof *arg;
    printf ("arg_len=%zu\n", len);
    strncpy (*arg, "modified a bit more", len);
}

int main (int argc, char* argv[])
{
    entity_id arg = {'\0'};
    fill_argument_safely_and_elegantly (&arg);
    printf ("arg has: %s\n", arg);
    return 0;
}
{% endhighlight %}

Result:

{% highlight console %}
arg_len=32
arg has: modified a bit
{% endhighlight %}

Success!

Not only I didn't have to pass another argument around but the compiler knows
exactly how large the size of 'entity_id' is.

It's also obvious now that the function is not receiving a copy of anything. 
The signature clearly shows that 'arg' is a pointer to an 'entity_id', so 
there is no confusion there either.

Why does this work? Because the compiler treats this as:

{% highlight C %}
void fill_argument_safely_and_elegantly (char (*arg)[32])
{% endhighlight %}

It now knows to receive a pointer to an array of 32 characters **exactly**,
if you try passing anything else it will fail at compile time.

For instance, writing this:

{% highlight C %}
char buf[33];
fill_argument_safely_and_elegantly (&buf);
{% endhighlight %}

...will have the compiler saying:

{% highlight console %}
app.c:34:5: warning: passing argument 1 of ‘fill_argument_safely_and_elegantly’ from incompatible pointer type [enabled by default]
app.c:19:6: note: expected ‘char (*)[32]’ but argument is of type ‘char (*)[33]’
{% endhighlight %}

Which is exactly what you'd want.

# Should I use typedef'd arrays?

If you can, no.
All the source of confusion we started with are enough arguments against it.

The tips presented are helpful, though, if you absolutely must live with source
code that is littered with them, in which case understanding how they behave
and how they should be used is a must.

A good alternative would be to define a structure to hold the array instead:

{% highlight C %}
typedef struct
{
    char id[32];
} entity;
{% endhighlight %}

This will make it much easier to understand and follow the code even when
you don't have the definition in front of you.

It also has the added benefit that it makes your code extensible, when
the day comes that you need to associate something else to your entity.


