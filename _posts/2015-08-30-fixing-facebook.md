---
layout: post
title: Fixing Facebook
category: Random
---

Have you been thinking Facebook's newsfeed sucks lately? I sure have. It's been
bad for a few years now, but in the last few months it's gotten unbearable.
Even worse is the new "Trending Stories" module, which seems to have been
inspired by the New York Post. I started wishing, if only I could just get rid
of them without losing Facebook's other functionalities...

Then I remembered. I CAN do that.

Browsers allow users to upload custom stylesheets, and Facebook's ids are
unique enough to avoid conflicts with other websites. So regardless of your
browser choice, all you need is a few lines of CSS to vanish the newsfeed and
trending stories for good (well, until the next time Facebook changes the ids
and the stylesheet needs to be updated).

{% gist bd1a515bc7f4000a92a8 %}

###To Use:

####For Safari:
* Open the 'better\_facebook.css' link under the box above
* Click "Download ZIP" on the right
* Open/unpack the downloaded ZIP file
* Move the 'better\_facebook.css' file from the unzipped folder to somewhere safe
* Open Safari and click Safari > Preferences > Advanced
* For "Style sheet" choose "Other..." and select the 'better\_facebook.css' file

####For Chrome/Firefox:
* Copy the contents of the box above
* Download the Stylish extension
([Chrome](https://chrome.google.com/webstore/detail/stylish/fjnbnpbmkenffdnngjfgmeleoegfcffe)
or [Firefox](https://addons.mozilla.org/en-us/firefox/addon/stylish/))
* Click the Stylish icon > "Manage installed styles" > "Write new style"
* Paste the text you copied into the "Code" box
* On the left, enter "Better Facebook" or a name of your choosing
* Click "Save"

When you return to Facebook, it should be beautifully free of the newsfeed and
trending stories. You'll notice that you can still scroll on the homepage -
this is because hiding the newsfeed entirely causes it to expand infinitely
while invisible, which is spectacularly unhelpful.

Now ignore Facebook, and go find some real, non-clickbait news!
