---
layout: post
title: Introduction to Object-Oriented Programming
category: Back to Basics
---

Although the first week of the Metis Ruby on Rails course has been much less intense than I'd expected, the first week has been a somewhat jolting dive into object-oriented programming. Before the class started, the bits of Ruby I'd done were just lines of code. I thought classes were for CSS, methods were just words that did cool things to variables, and objects were for special uses of Python or Java.

Arrays, hashes, strings, numbers, variables, and performing basic functions on them are the easy bits. I've had a lot more difficulty wrapping my head around objects and what to do with them.

We jumped into using objects on Day One, when we created a simple guessing game. The computer picks a random number between one and ten, and you try to guess what it is (until you get it wrong three times). Then we added the ability to play multiple rounds, and to end the game by telling you how many times you won, as well as the depressingly-close-to-three average number of guesses per round.

This game uses at least a Game class and a Round class, and multiple methods, or chunks of useful code, within each of them. There's only one game, or one game object, each time you run the program, but there are multiple rounds, or multiple instances of the round object. Or more, multiple round objects. The Game class must introduce the game, run the rounds, count the total scores, and announce the scores at the end. This means that the class should contain at least a welcome method, a loop that creates and starts each round, a scoring method, and an ending method.

The Round class, meanwhile, must choose a random number, ask for guesses, tally guesses to be calculated in the average, evaluate guesses, and respond. It gets to have a method for each step, and a loop that runs the guess/tally/evaluation/response pieces three times, unless the loop is broken by a correct guess. (Which then starts a new round, unless the game has already looped through the Round class five times.)

After my initial dismay about cutting a program into pieces, the part I struggled with was understanding how the rounds worked as objects, and more specifically, getting the two classes to communicate on the guess count. Having the game count and total the wins on its own was simple, but the game kept counting only one guess for each round (definitely not accurate). In the end, the solution was adding a method to the Round class that simply pointed to the variable collecting the guess tally, and then, in the loop within the Game class, calling said method on each instance of the Round class.

On the first day, the idea of having an object (a class) that's also multiple objects (each instance), was incredibly confusing. A few days later, it makes a whole lot more sense, though I still have to fight my inclination to keep the numbers of classes and methods as small as I can get away with. It's only been a week though, and I've come a pretty long way. My little [music database solution](https://github.com/HayleyCAnderson/WeekOne/blob/master/music.rb) has six whole methods, accurate names, and appropriate spacing that I did entirely without help or pointed suggestions!

I'm excited to have started looking at data, and I can't wait for next week, when we start working with SQL, HTTP, gems, and Sinatra. I think having a less limited view of the use of code will help, and I'm eager to start moving towards work that has more real-life applications. Check back next week, and I'll try to improve my blog game as well as my distaste for classes.
