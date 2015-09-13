---
layout: post
title: Running on Rails
category: Back to Basics
---

I'm going to start aiming for shorter, more frequent posts. The last couple
weeks have been pretty overwhelming, as we started learning Rails. There is
basically just so much information, I don't know where to start. Regardless,
Rails is great. I didn't expect to like it so much. But this weekend I've been
at a hackathon where I was asked to work on a Bootstrap and a Wordpress site,
and I desperately missed the organization and structure of Rails.

Rails is an MVC, or model-view-controller framework. This means that it's
structured with separate sections for models, views, and controllers, and these pieces work together to determine how the application runs.

Models get information from the database and determine how the database tables relate to each other. This means, for example, that if your application has users and groups, the models can relate the users table and the groups table so that users can join groups.

Views are more like the actual pages of the application. There might be views
for an index page, a user page, a group page, pages with lists of all the users or groups, pages for making new users or groups, etc.

Controllers connect the models and views, and fetch the information from the
model that will be required for the view. It might say that for the user view,
you need to get information about the user (from the model) before showing the
user page.

In the end, when a user navigates through your application, each request they send, by clicking on a link, submitting a form, etc., will go to the routes. The routes determine which pages and actions your application will have. The routes will send the request to the corresponding section of the controller, which fetches the information it needs from the model, which connects with the database and sends back information accordingly. The controller then sends the information to the corresponding view, which renders the correct page and allow the user to continue navigating through the website and
sending requests.

So that's the basics of Rails. It's a very cool framework, and I'm excited to
start working on bigger and more complex Rails applications, and writing about
some of the more interesting things that Ruby and/or Rails can do.
