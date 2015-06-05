---
layout: post
title: "Open source doesn't mean open use"
description: "Understanding how open source code can and cannot be used."
tags: [Open Source, Legal]
#comments: true
#share: true
---

Did you ever ask yourself if you can *actually* use that open source code, 
library or framework your application uses?

Even to this day a lot of people assume that because something is OSS (Open Source Software)
you can use it freely and with no limitations.

Tread carefully.

Whether this is true or not depends on how your software is used, and how
it will use the 3rd party code.

Before I go any further I must say this: I am no expert on the matter. I certainly **am no lawyer**.

The purpose of this post is just to give you a *very* quick overview of your options and,
really, raise your awareness. If you need professional advice, please consult
with, a **legal** professional in this area, for which I'm not.

With that out of the way, let's see some of the most commonly used licenses
released with OSS out there.

# Licenses

Typically you will find that public source code repositories are accompanied
by a license file. For example, Github makes it easy for you to create a LICENSE file
by choosing one of the most common types out there.

The license will describe if the software in question can be used freely or not,
and under which conditions. You will find some are more permissive than others. 

Chance are you will have come across at least the following 4 types of licenses.

## GPL

Under the GNU General Public License (GPL) there are several variants
and how restrictive they are depends on how the software is integrated with yours.

Common across all of them, though, is the fact that typically, while you
can use the OSS commercially, you have to release *your* code publically under the same license.

Right away you see this may be a blocker if you are writing software for
which you do not wish to release the source code.

There's a good example of this happening. Linksys, as part of their WRT54G product,
used open source Linux components. The community spotted this
and, because Linux is released under GPL, requested Linksys to release the entire source code
of their product out for the world to see.

Take care if you don't want to be caught in the same situation.

## Apache

Apache licenses are more permissive than GPL. 

As long as you provide attribution,
a full copy of the license and you take note of the changes you've done
to the original then you can use the code in your software, even commercially,
with no need to release your own source code.

## BSD, MIT

The BSD and MIT licenses are even less restrictive and allow you to use the OSS
freely as long as attribution and a copy of the license is released with it.

There are subtle differences between the two which you should analyse more carefully
for your own needs.

# Dual, or multi-licensing

In some circumstances you may find a piece of OSS released under more than one license.
This gives you the freedom to choose which fits you best.

Common cases for this are distinct licenses to use under commercial and open source use,
depending on your intent.

# No license != Free use

If the author of the software forgot to release a license along with it
then all copyright is reserved. You cannot use it before first ensuring
the author will allow you to use it in his/her conditions.

The ommission is probably an indication that the author never thought much
about it, not that the software is free to use with no restrictions.

If you have released open source software in this manner then take the time
to consider this and please specify a license that fits your intention to avoid
any doubts.

# Conclusion

If you need an "executive summary" of each type of license you can use sites
like [tl;dr Legal](https://tldrlegal.com/) to give you an overview of what each enforces
and allows you to do or not.

When in doubt, seek professional legal advice. Only a professional will be able
to tell you exactly what your license needs to look like for your software and 
how you intend it to be used.

Most importantly, realise that OSS doesn't mean free. That will depend on the
license it's been released with and how you use it.

Also, be a gentleman. Even if you find OSS released with little to no restrictions,
and doesn't force you to give attribution, there's nothing wrong with giving them
the proper kudos for the effort they had in making it available to you.


