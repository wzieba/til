# How Glide tags images and how to get those tags?

## Problem
I would like to know if Glide performed operations on ImageView without need to compare images (which might result with tests flakiness).

## How to do this?
Glide assings `tag` on each ImageView it inflates. There are two ways one can do this in Android
- general purpose tag: `ImageView#setTag(object)`
- keyed tag: `ImageView#setTag(key, object)`

In previous versions Glide used the first one. This resulted with 

> You must not call setTag() on a view Glide is targeting" on non-root ImageView

error. One can find lots of issues on StackOverflow or Github Issues complaining about this.

In some version between 4.2.0 and 4.8.0 (couldn't find this in release notes), Glide moved from setting general purpose tag to keyed tag with, thankfully, exposed `R.id.glide_custom_view_target_tag` id for referencing the tag.

After getting this tag, one can see it's `SingleRequest`. It contains e.g. image URL address, but what's most important, is proof that Glide peformed image inflation on this ImageView

