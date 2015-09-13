---
layout: post
title: Sometimes It's Not Your Fault
category: Back to Basics
---

My blog has been pretty dead for a while, as I've been busy with starting a new
job, moving, learning how to be an adult, etc. But that's not the reason I first
stopped posting on the blog - it was because I was questioning everything I was
saying. My [pet project](/2014/12/24/using-the-socrata-open-data-api-and-gem-to-make-complex-queries)
was falling apart with problems I couldn't
explain, and I thought I'd messed it up terribly.

One of the first things you learn when starting to code is that it's
your fault when things go wrong. If there's an error, it's because you did
something wrong, not because the computer randomly decided to screw with you.
But then there's no fanfare to tell you later on that even programming languages
and operating systems were made by humans.
They're buggy or badly planned or don't act the way they should, and when things
go wrong, it's not necessarily just because you're a clueless beginner who
messed things up.

I spent all of January and half of February trying to figure out why [SaferNYC](http://safernyc.com)'s
memory usage was so out of control that its Heroku dyno kept crashing.
The memory levels and garbage collection were wildly erratic and out of control.
With the amount of data and GeoJSON the application deals with, it's not
surprising that the memory would be somewhat high, but it was 3 or 4 times the
memory of a typical Rails app. One dyno should have been plenty, and it's all
that's affordable for a side project, but SaferNYC was running above the 512MB
limit even with no traffic.

I couldn't figure out how I'd created such a massive memory leak without
noticing. And worse, I couldn't even find it after the fact. I eliminated every extra
variable instantiation I could find, got rid of gems, tried to test where the
memory was going - not to any objects I was making, as far as the tests seemed
to say. I researched and researched, asked more experienced
engineers, and all I got was that it must be a memory leak that I'd created.

After weeks of research, begging for help, and making a hacky fix with the
garbage collector, I
ran across a message board saying that some Ruby versions didn't garbage collect
properly. I read it and updated my Ruby version, but I didn't believe that could
actually be the problem, and went right back to searching for the leak. It
wasn't until later that I decided caching would solve all my
memory and performance problems, and replaced my garbage collector hack with a
caching call.

After deploying (before I'd set up staging), I was horrified to discover that not a
single item had been cached (the datasets were too large). And yet, when I
rushed to check, I discovered that the garbage
collection and memory levels remained steady despite the deletion of the call to
the garbage collector. I checked my commits - no, I hadn't made any other
back end changes.

It was the Ruby version. A month of trying to figure out what I'd done
so terribly wrong, and I'd done nothing wrong at all (at least not to cause this).
Ruby version 2.1.2 had a problem with garbage collection in some cases, and
upgrading to 2.1.3 solved the memory problem completely. As much as languages and
frameworks seem so advanced and untouchable, they're all created by humans.
Sometimes, it really isn't your fault.
