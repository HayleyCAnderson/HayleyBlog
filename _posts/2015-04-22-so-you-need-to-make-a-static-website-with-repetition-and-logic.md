---
layout: post
title: So You Need to Make a Static Website with Repetition and Logic
category: Random
---

Or When and How to Use Middleman
----------
Maybe you've made static websites, maybe you've made Rails applications, but what
happens if you need to make something that's in between? It's not a dynamic
application, but it will require repetitive code, use logic, or
change frequently enough to make changing the code by hand a pain.
<strong>The solution? Possibly
[Middleman](https://middlemanapp.com)</strong>.

Middleman is a Ruby-based static
site generator that's reminiscent of Rails and keeps its opinions mostly to
itself. Static site generators can handle logic and cut repetition, but they
build into static pages that are much faster and easier to host than
applications.

There are many other well-known options,
such as Octopress and Jekyll (which this blog is on), but most are intended primarily for
developer blogs. It's certainly possible to use most such generators for more,
but their main purpose tends to be overly clear.

Middleman is more of a blank slate, at least to a Rails developer. It feels
like using a lightweight version of Rails, complete with access
to many of the basic helper methods available in Rails. It comes with built-in support
for ERb, Haml, Sass, SCSS, and CoffeeScript.

Most importantly, Middleman makes it easy to add data and helpers. Data files allow you to
add data or content using YAML or JSON. This data can then be accessed from the
views and/or config file. Meanwhile, helper methods in the
config file or in separate modules can perform logic in Ruby that can be used in
the views.

Use Data, Variables, and Proxies
----------
With Middleman, you can generate repetitive pages based on
[JSON data](https://middlemanapp.com/advanced/data_files/) by adding a
proxy to the `config.rb` file:

{% highlight ruby %}
# Creates person_page for each person
data.company.people.each do |person|
  proxy "/#{person.id}.html", "/person_page.html", locals: {
    id: person.id,
    name: person.name,
    photo: person.photo,
    bio: person.bio
  }
end
{% endhighlight %}

Then in `person_page.html.erb`, you can take advantage of the locals provided by
the proxy:

{% highlight erb %}
<div id="person-info">
  <h1><%= name %></h1>

  <%= image_tag("images/#{photo}") %>

  <h2>Bio</h2>
  <%= bio.html_safe %>
</div>
{% endhighlight %}

No matter how many people you have or edits you make, you only need to change
`person_page.html.erb` or the corresponding JSON, but when you create the build,
a file will be generated for each person, named using their id, with their name,
photo, and bio filled in.

Perform Logic in Helpers
---------
Adding links to each page to allow someone to continuously flip through a set of
people pages requires a bit of logic, which is where being able to add helpers
to the config file is incredibly important.

{% highlight ruby %}
helpers do
  # Links back button to previous person
  def previous_id(current_id)
    if current_id == 1
      data.company.people.length
    else
      current_id - 1
    end
  end

  # Links next button to next person
  def next_id(current_id)
    if current_id == data.company.people.length
      1
    else
      current_id + 1
    end
  end
end
{% endhighlight %}

But Beware
----------
Middleman is not
automatically set up for relative linking, and if you're not prepared for that,
you'll be in for a rude surprise when you make your build and find yourself without
styling, images, or linking between pages.

To allow relative links pointing to
assets - stylesheets, javascripts, images, etc. -
add `activate :relative_assets` to the build configuration in the
config file. To enable relative linking elsewhere, add `set :relative_links,
true` to the config file (outside the build configuration). Also, you'll
need to use `link_to` helpers instead of just `a href`.

Although it seems a bit odd to have `link_to` but not relative linking,
Middleman is pretty generous with the helper methods. Make sure to
check out the [full list](https://middlemanapp.com/basics/helper_methods/)
because you may be surprised by what's available.

And if you need something more, Middleman seems like the type of framework where
the sky is the limit. At least as long as you're still just looking for a static site
generator.
