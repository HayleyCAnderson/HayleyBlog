---
layout: post
title: Strings and Arrays and Linked Lists, Oh My!
---

I've spent some time the past couple weeks learning about data structures and algorithms. I started out [here](http://code.tutsplus.com/tutorials/algorithms-and-data-structures--cms-20437), reading about some of the basics of data structures. I've since been reading [Problem Solving with Algorithms and Data Structures](http://interactivepython.org/runestone/static/pythonds/index.html) on Interactive Python, and everything is starting to make more sense.

Strings, arrays, and linked lists are a few types of data structures - objects used to store information. They're used in algorithms, which are sort of like the underlying processes behind programs. Programs are created to make certain things happen, but algorithms are more general. They focus more on the details of how data is stored and manipulated. As a result, one of the main concerns with algorithms is their effectiveness in terms of speed and memory use.

Speed of an algorithm is represented by Big-O (order of magnitude) notation. Big-O notation is shown as O(function of n), with n being the size of the problem - for example, if 5 items are being iterated over in the algorithm, n is 5. Since algorithms are not meant for particular situations, n is sometimes treated like the limit to infinity. This means that f(n) is often simplified so that most algorithms give just a few functions for Big-O notation.

Some of the common functions are constant, O(1), logarithmic, O(log(n)), linear, O(n), log-linear, O(nlog(n)), and quadratic, O(n<sup>2</sup>). If you look at these functions on a [graph](http://interactivepython.org/runestone/static/pythonds/AlgorithmAnalysis/BigONotation.html), you can see that they're in order of increasing slope. Algorithms with functions of lower slope, such as log(n), are typically preferred over algorithms with functions of higher slope, such as n<sup>2</sup>, because they are less likely to take an unacceptable amount of time if the dataset grows very large.

A constant algorithm takes the same amount of time regardless of the size of the problem - maybe the algorithm simply replaces the first item in an array without regard to the other items in the list. A linear algorithm may have a single loop that loops through a list of n items once, so as n increases, the time required linearly increases.

A logarithmic algorithm may also have a loop, but the loop may break or be limited so that the amount of time required does not increase as quickly as the number of items increases. On the other hand, a quadratic algorithm could have a loop looping through each item <em>within</em> the first loop. In this situation, the time required obviously increases very quickly as n grows very large, and it suggests that another algorithm may be preferable.

I'm still learning how to actually write algorithms, but last night I was able to watch a practice technical interview through [Lady Storm Hackathons](https://www.facebook.com/groups/LadiesStormHackathons/), which helped a lot in elucidating the mysterious coding interview. With some more practice, I hope to catch up with some of the computer science knowledge I've missed in time to do well in coding interviews.