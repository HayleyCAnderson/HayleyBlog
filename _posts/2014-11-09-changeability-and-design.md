---
layout: post
title: Changeability and Design
category: Back to Basics
---

I always come up with ideas for posts, but then I never feel like actually writing the posts. So now I have to combine several of the topics I was intending to write about, including git and polymorphism. They do, after all, have the same themes in common. In short, they can provide, and are best used with, carefully planned design and an ability to smoothly make changes as time goes on.

Git can be a rough thing to learn (see [my post](http://blog.hayleyanderson.us/2014/10/03/github-and-the-magical-black-hole-of-version-control/)), but now I'm really starting to see the beauty in it. You can make concise, sensible commits, and then later remove the ones that introduced problems without losing other work. You can make branches to designate different features, types of work, or contributors, then loop them back together and later see how you got to where you are now. But it all rests on designing your workflow and planning for the future.

Code is the same. Design, plan, break it up into smaller parts, leave it flexible for future changes. One of the coolest examples of this is polymorphism. Partly just because of its awesome name. But also because it's a really awesome tool. Polymorphism can prevent both repetition and conditionals, and it's an important technique that can be used in designing code to be flexible.

Polymorphism can be used in different ways, but in general, it allows pieces of code to run differently depending on the situation. Rather than hard-coding a method to take a particular variable, you can pass it an argument that may change - because your code is changing frequently, the variable may be different depending on the situation, or you'd like it to accept multiple different pieces of information. Polymorphism, by definition, makes your code changeable - in exactly the way you need it to be.

The best part about this learning experience is starting to see things come together and being able to spot and appreciate the benefits of good design and planning. Programming is starting to seem more like an art, and I finally start to get why people complain about certain types of bad programming and all the git tutorials spend more time than anything warning that you're going to anger other developers. Maybe I can't program well, but at least I know what they're talking about. Appreciating changeability and design - it's a start.

For more on polymorphism (via thoughtbot):<br>
* [Back to Basics: Polymorphism and Ruby](http://robots.thoughtbot.com/back-to-basics-polymorphism-and-ruby)<br>
* [Refactoring: Replace Conditional with Polymorphism](http://robots.thoughtbot.com/refactoring-replace-conditional-with-polymorphism)<br>
* [Using Polymorphism to Make a Better Activity Feed in Rails](http://robots.thoughtbot.com/using-polymorphism-to-make-a-better-activity-feed-in-rails)
