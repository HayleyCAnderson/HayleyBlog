---
layout: post
title: Creating a Django Widget to Draw and Calculate Dimensions of Rectangles on Images
category: Django
---

In my [last post](/2015/07/26/passing-information-between-django-form-wizard-steps/)
I talked about getting an image path and passing it into the next step of a
Django form wizard for use in the `canvas` field. That was only the first piece
of the challenge. The purpose of the `canvas` field is to hold a widget
containing a canvas to allow users to draw a rectangle within the image. The
placement and dimensions of the rectangle - specifically distance from top and
left, and height and width, as a percentage of the image, need to be entered
into the database to create a positioned 'hotspot'.

First, I found a canvas drawing plugin that would provide crosshairs for
rectangle drawing functionality. In a world of disappointing options,
[wPaint](https://github.com/websanova/wPaint) seemed like the best/only choice.
Then I created a widget with the wPaint dependencies added as media.

{% highlight python %}
class HotspotDrawingWidget(Input):
    ...

    class Media:
        css = { 'all': [
            'creator/lib/wColorPicker.min.css',
            'creator/wPaint.min.css',
        ] }

        js = [
            'creator/lib/jquery.1.10.2.min.js',
            'creator/lib/jquery.ui.core.1.10.3.min.js',
            'creator/lib/jquery.ui.widget.1.10.3.min.js',
            'creator/lib/jquery.ui.mouse.1.10.3.min.js',
            'creator/lib/jquery.ui.draggable.1.10.3.min.js',
            'creator/lib/wColorPicker.min.js',
            'creator/wPaint.min.js',
            'creator/plugins/main/wPaint.menu.main.min.js',
            'creator/plugins/text/wPaint.menu.text.min.js',
            'creator/plugins/shapes/wPaint.menu.main.shapes.min.js',
            'creator/plugins/file/wPaint.menu.main.file.min.js',
        ]
{% endhighlight %}

This allows widget media dependencies to be loaded in dynamically through the
wizard template or manually through the widget render method. I loaded them in
manually when creating the plugin div and loading the custom JavaScript for
calling and modifying the plugin.

Although I was specifically looking for a plugin that could have an image as the
canvas background, wPaint's image functionality did not work. Instead, I loaded
in the image within the wPaint div and gave it 100% height and width. This
caused problems in Safari, but it worked in Chrome. The requirement of this
application is only that it works in Chrome, so this was fine.

Here's the
[widget's render method](https://docs.djangoproject.com/en/1.8/ref/forms/widgets/),
with the image path being found in the `value` argument, which is the value of
the widget's field in the form's initial dictionary:

{% highlight python %}
class HotspotDrawingWidget(Input):
    def render(self, name, value, attrs=None):
        # Image path passed in as value through initial dict
        image = '<img src="%s" id="canvas-bg-image" style="height: 100%%; width: 100%%;">' % value

        markup = '<div id="wPaint" style="position: relative;">%s</div>' % image
        script = '<script src="%screator/hotspot_drawing_widget.js"></script>' % settings.STATIC_URL

        # Returns CSS/JS requirements, wPaint div, and custom widget JS as render
        return str(self.media) + markup + script

    ...
{% endhighlight %}

Now, we just need to call the plugin in the hotspot drawing widget script, and
the canvas functionality will appear:

{% highlight javascript %}
// Call drawing plugin in rectangle mode with custom colors
$("#wPaint").wPaint({
  mode: "Rectangle",
  fillStyle: "#D6EDF5",
  strokeStyle: "#D6EDF5"
});
{% endhighlight %}

Unfortunately, the plugin gives the user about a million options for how to
draw, and we don't want them to have choices. The wPaint docs claim you can
enter the options you want as arguments, but that doesn't actually work, so
we'll just remove the buttons and resize the menu on load.

{% highlight javascript %}
// Remove unwanted menu buttons
$(".wPaint-menu-icon-name-rectangle").remove();
$(".wPaint-menu-icon-name-ellipse").remove();
$(".wPaint-menu-icon-name-line").remove();
$(".wPaint-menu-icon-name-pencil").remove();
$(".wPaint-menu-icon-name-text").remove();
$(".wPaint-menu-icon-name-eraser").remove();
$(".wPaint-menu-icon-name-bucket").remove();
$(".wPaint-menu-icon-name-fillStyle").remove();
$(".wPaint-menu-icon-name-lineWidth").remove();
$(".wPaint-menu-icon-name-strokeStyle").remove();
$(".wPaint-menu-icon-name-save").remove();
$(".wPaint-menu-icon-name-loadBg").remove();

// Resize menu to fit remaining buttons
$(".wPaint-menu").css({width: "auto"});
{% endhighlight %}

Now there's a canvas over the user-chosen image, and we can draw rectangles,
undo, redo, and clear the canvas. That's exciting, but how do we find out what
the user drew and pass that data back into the form so it can be entered in the
database as percentages of the image?

While inspecting the canvas, I was thrilled to discover that the plugin provides
a div called 'wPaint-canvas-temp' that has height and width, and inline CSS for
top and left, in pixels, representing the last drawn rectangle. Finally, the
plugin does something I want! Except, the div changes as the user draws, and the
user can also undo, redo, and clear. How do I accurately grab that information
on save?

The JavaScript devs I asked for suggestions were bewildered by the idea of
passing information to a back end, so I decided to focus on acting for the
benefit of what I care about, which is the back end. I could have possibly made
a JavaScript function to run when a user clicked the submit button, but that
struck me as risking the integrity and reusability of the back end.

So, I wanted to create inputs for the top, left, height, and width fields
already associated with the hotspots model, and have their values change as the
canvas changes. That way, there's no worries about syncing the back and front
end, everything is done within the widget, and the database receives the
positioning values as if they were manually entered by a user in a normal form.

I added both initial and default values of '0%' for each of the fields, and
updated the form for the canvas page, with Django-provided `HiddenInput`
widgets:

{% highlight python %}
class HotspotDrawingForm(forms.ModelForm):
    # Fake field to hold drawing widget
    canvas = forms.CharField(
        required=False,
        help_text='Draw exactly one rectangle inside the image.',
        widget=HotspotDrawingWidget
    )

    # Hidden inputs filled by JS and then entered directly into database
    top = forms.CharField(widget=HiddenInput)
    left = forms.CharField(widget=HiddenInput)
    height = forms.CharField(widget=HiddenInput)
    width = forms.CharField(widget=HiddenInput)

    ...
{% endhighlight %}

I used the [jquery-watch](https://github.com/RickStrahl/jquery-watch) plugin to
watch changes to the outer HTML of 'wPaint-canvas-temp'. Users should not be
spending much time on this page, so performance shouldn't be, and so far hasn't
been, an issue. When changes are detected, a function is run to pull out the
top, left, height, and width, calculate percentages, and insert into the
associated inputs.

After adding 'creator/jquery-watch.min.js' to the media to be loaded, I added
the watch function and calculation function. It's ugly, but functional:

{% highlight javascript %}
// Monitor changes to outer HTML of temporary canvas (most recent drawing)
$(".wPaint-canvas-temp").watch({
  properties: "prop_outerHTML",
  watchChildren: true,
  callback: function (data, i) { setValues(String(data.vals[i])); }
});

// Calculate percentages of new drawings as compared to image
// Add percentages as values of hidden inputs
function setValues(outerHTML) {
  var top = /top: (\d*)px/.exec(outerHTML)[1];
  var left = /left: (\d*)px/.exec(outerHTML)[1];
  var height = /height="(\d*)"/.exec(outerHTML)[1];
  var width = /width="(\d*)"/.exec(outerHTML)[1];
  var canvasHeight = $("#canvas-bg-image").height();
  var canvasWidth = $("#canvas-bg-image").width();
  $("#id_1-top").val(String(((top / canvasHeight ) * 100).toFixed(2)) + "%");
  $("#id_1-left").val(String(((left / canvasWidth ) * 100).toFixed(2)) + "%");
  $("#id_1-height").val(String(((height / canvasHeight ) * 100).toFixed(2)) + "%");
  $("#id_1-width").val(String(((width / canvasWidth ) * 100).toFixed(2)) + "%");
};
{% endhighlight %}

And finally, everything works. Users can choose a page and surface to add a
hotspot to, view the corresponding image and draw on it where they would like to
place the hotspot,
[upload an image or video](/2015/07/18/validating-file-types-in-django/) if it
is a popup-style hotspot, save their hotspot, and use it in a finished
presentation.

<iframe src="https://docs.google.com/file/d/0B5ECX-9TWH-CdTU1Mm1CVHlxNGc/preview" width="100%" height="480"></iframe>
