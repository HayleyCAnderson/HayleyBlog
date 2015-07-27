---
layout: post
title: Passing Information Between Django Form Wizard Steps
---

For the project I'm working on, users need to be able to choose a page and
surface and then view the image corresponding to that page and surface and add
a hotspot to it. Since this requires hitting the database to get the requested
image path, I decided to place these steps on different pages using the
form wizard from Django's
[form tools](http://django-formtools.readthedocs.org/en/latest/index.html)
package. As with many things Django, passing information from one step to a
later step where it will be used is a seemingly obvious but relatively
undocumented use case.

Getting step data from an instance of the wizard appears simple, but as I went
to actually implement it, I realized... where is the instance of the wizard
anyway? Setting up the wizard actually does not involve explicitly instantiating
an instance of the wizard - nor of any of the forms, for that matter. The wizard
instance can in fact be found in the wizard template file, but not only is that
a bad place for accessing and manipulating data, but it could not (sensibly, at
least), be passed from there back into the appropriate form instance.

The next option is to access the wizard instance from within the wizard's
existing instance methods. Fortunately, there's one that's going to
automatically fix your next problem - that of getting the information back into
the appropriate form. The wizard instance method `get_form_initial(self, step)`
is going to be called on each form/step within the wizard form and provide to
that step a dictionary of initial values presumably corresponding to each form
field.

It did not occur to me at first to make use of the initial dictionary (despite
[this](http://chriskief.com/2013/05/24/django-form-wizard-and-getting-data-from-previous-steps/)
being the only relevant post I could find) because I didn't have any form fields
that I wanted to be given defaults to show to the user. But guess what, there's
no requirement to use the initial dictionary in this way -
this is Django! I already had a fake field on my hotspot form to hold the widget
that will display the image to the user, so it may as well have a fake initial
value too. And in fact, thanks to the way widgets work, using the initial
dictionary is by far the easiest way to pass in data.

So how do initial dictionaries work? Here's the
[original method](https://github.com/django/django-formtools/blob/master/formtools/wizard/views.py#L374),
on `WizardView`, which is the parent class of `CookieWizardView`:

{% highlight python %}
class WizardView(TemplateView):
    ...

    def get_form_initial(self, step):
        """
        Returns a dictionary which will be passed to the form for `step`
        as `initial`. If no initial data was provided while initializing the
        form wizard, an empty dictionary will be returned.
        """
        return self.initial_dict.get(step, {})

    ...
{% endhighlight %}

And [source code commentary](https://github.com/django/django-formtools/blob/master/formtools/wizard/views.py#L135)
on `initial_dict`:

> `initial_dict` - contains a dictionary of initial data dictionaries. The key
should be equal to the `step_name` in the `form_list` (or the str of the zero
based counter - if no `step_names` added in the `form_list`)

Somewhat confusing, but very useful. And then here is my version of the method:

{% highlight python %}
class HotspotWizard(CookieWizardView):
    ...

    def get_form_initial(self, step):
        initial = {}

        # If at second step, add image path to initial data for canvas field
        if step == '1':
            first_step_data = self.storage.get_step_data('0')
            page_id = first_step_data.get('0-page','')
            surface = first_step_data.get('0-surface','')
            initial['canvas'] = self.get_image_data(page_id, surface)

        return self.initial_dict.get(step, initial)

    def get_image_data(self, page_id, surface):
        page = Page.objects.get(id=page_id)

        if surface == 'brick':
            return page.brick_background_path
        else:
            return page.glass_background_path

    ...
{% endhighlight %}

The method takes the name of the current step, an integer in string form,
starting at zero. If the current step is the second step, then the data from the
first step is fetched and is used to find the corresponding image path. The path
is then entered into the `initial` dictionary with the key being `canvas`, the
name of the fake field which will need access to the path. The method returns
the already existing initial dictionary for the step in `initial_dict`,
or, since that key-value pair shouldn't already exist, the `initial` dictionary
created within this method.

This means that on any step besides the second step, the initial dictionary will
be the default, an empty dictionary, and on the second step, it will contain a
value for the `canvas` field. It's important that the value be for the canvas
field, because the canvas field is represented by a
[widget](https://docs.djangoproject.com/en/1.8/ref/forms/widgets/), which is
where the path needs to be used. A widget is rendered by the method
`def render(self, name, value, attrs=None)`, and `value` is the value corresponding
to the field key in the initial dictionary for the current form. Thus, `value`
is the image path, and the image can now be easily rendered through the widget.
