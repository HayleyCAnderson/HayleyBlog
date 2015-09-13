---
layout: post
title: Validating File Types in Django
category: Django
---

So, I recently picked up Python, and then Django, because expanding my world
view or something. Mostly, it's been awesome. And a bit of an adventure. When
you dream up some crazy thing you're trying to implement in Rails, you can
usually just google your stream of conciousness and someone serves you up a gem
on a silver platter and tells you exactly (more or less) how to use it. With
Django, it's just the wild, wild west out there, even when you're doing something
you'd think would be very simple and common. Uh, good luck!

Django does have a lot of great things built into it to make up for the lack of
associated packages, and one of those things is file uploads. Put a 'FileField' on
your model, and now users can upload files to their hearts' content! Problem is,
now they can happily upload some 2GB PowerPoint or 'evil.jar'. You probably don't
want that. There's an extension of 'FileField' called 'ImageField' that will solve
all your problems if you only want users to upload images, but you're on your
own if you need something in between.

During my first round trying to solve this problem, the wild, wild Django
community recommended writing a new extension of 'FileField' to have the same
functionality as 'ImageField' but accepting the file types of your choice. They
and the docs suggested reading the source code to figure out how to actually do
this. Since I'm gullible, I can tell you that the source code is actually pretty
good - but this is silly. Django actually does have validation functionality
somewhat similar to Rails, if only those on Stack Overflow could remember it.

First, you're probably going to want to make a new file called 'validators.py' -
but this is Django, so you do you. Then run `brew install libmagic` (on Mac with
Homebrew) and `pip install python-magic`.
[Magic](https://github.com/ahupp/python-magic) is going to identify your file
types. You're going to need to import that and some other modules you already
have access to, and make a rogue method that's going to become your validation:

{% highlight python %}
import os
import magic
from django.conf import settings
from django.core.files.storage import default_storage
from django.core.files.base import ContentFile
from django.core.exceptions import ValidationError

def validate_file_type(upload):
    ...
{% endhighlight %}

And what is the 'upload' being passed into the rogue method? That's going to be
the field you've placed the validation on, in this case 'FileField'. Meaning, if
you have a model called 'Popup' with a 'FileField' called 'content', 'upload'
will represent `popup_instance.content`, and you can access the uploaded file
at `upload.file`.

So you can just access the file, use Magic to check the file
type, and be done, right? Well, no. You don't even want this file uploaded on
your server before it's been validated. So before validation, the file only
exists in memory and without an accessible path. You need to make it accessible
to Magic first, by temporarily saving it to the server. (This likely still
carries security risks, but that is beyond the scope of my current
internally-hosted project.)

First, name your file in a tmp directory - `upload.name` will return `./file_name`.
Then, read and save the file using Django's `ContentFile` and `default_storage`.
Finally, get the path, which will include the `MEDIA_ROOT` from your settings
file because of the `default_storage` functionality.

{% highlight python %}
def validate_file_type(upload):
    # Make uploaded file accessible for analysis by saving in tmp
    tmp_path = 'tmp/%s' % upload.name[2:]
    default_storage.save(tmp_path, ContentFile(upload.file.read()))
    full_tmp_path = os.path.join(settings.MEDIA_ROOT, tmp_path)

    ...
{% endhighlight %}

Now that you have an actual file saved somewhere and a full path that you use can
use to point Magic to it, you can check what the file type is and then delete it
before it can cause any problems or eat up your storage space.

{% highlight python %}
def validate_file_type(upload):
    ...

    # Get MIME type of file using python-magic and then delete
    file_type = magic.from_file(full_tmp_path, mime=True)
    default_storage.delete(tmp_path)

    ...
{% endhighlight %}

Finally, you can check the uploaded file type against your approved file types
and raise a validation error if it does not match. In my case, I wanted all
image types as well as all video types supported by Chrome. These are big, ugly
lists that I also need to access elsewhere, so I saved them as global variables
in my settings.

{% highlight python %}
def validate_file_type(upload):
    ...

    # Raise validation error if uploaded file is not an acceptable form of media
    if file_type not in settings.IMAGE_TYPES and file_type not in settings.VIDEO_TYPES:
        raise ValidationError('File type not supported. JPEG, PNG, or MP4 recommended.')
{% endhighlight %}

Once your validator is written, it's incredibly easy to use. First import it
in your models file - `from validators import validate_file_type`. Then, add it
to your FileField in your model:

{% highlight python %}
class Popup(Hotspot):
    ...

    content = models.FileField(validators=[validate_file_type])
{% endhighlight %}

Now the specified method will be run on your FileField so that your uploaded
files will be validated by file type. And since 'validators' is a list, you could
even make and add more! And have fun learning the internals of Django and trying
to ignore bizarre Stack Overflow suggestions while you're at it.
