---
layout: post
title: "Practicing what I preach"
date: 2018-01-25 09:14
comments: true
categories: a11y meta
---
I've finally updated the old blog a bit to be more accessible. It's
been on my to-do list for ages. Some of this blog is VERY old. It
started out on the Blogger platform in the early 2000s, moved to
my own hosted WordPress site for the latter 2000s, and has been on
GitHub pages for the last several years using
[Octopress](http://www.octopress.org).

That means the actual content has been through several layers of
import/export processes, and the resulting text is a mess. It
still is a mess, but it's slightly less of one now. When I did
the last import, I noticed that a lot of the content was VERY
dated, from ages past when I was a Java and C++ developer. That
being the case, I wasn't terribly excited to get it all looking
right, and left many things "good enough." That meant long broken
images, janky inline styles from old Wordpress plugins, and (of
course) lots of accessibility problems.

I think I've fixed most of the accessibility issues - the pages I
checked all passed a quick a11y audit using Google's excellent
[Accessibility Developer Tools](https://chrome.google.com/webstore/detail/accessibility-developer-t/fpkknkljclfencbdbgkenhalefipecmb?hl=en), and I think I checked all of
them. Some of the things I needed to do were:

* The colors on my links didn't have enough contrast. Google's
audit tool doesn't complain about links not having underlines
(I think it should, it helps people with color blindness and other
visual impairments identify what is a link and what is normal
text), so I added those in the main text.
* I used a really old Wordpress tool to generate my code samples,
and they all used inline stlpes for syntax highlighting. Those
inline styles didn't have enough contrast, either. I had to go
through the horribly mangled code samples and replace them with
the equivalent markdown to get them to pass.
* It seems the images imported from Wordpress either didn't have
alt tags, or they had alt tags that were blank. Google's audit
tool doesn't identify img elements with blank alt tags, with good
reason - a blank alt tag is actually the preferred way to indicate
that there is no applicable text representation of an image. But
that's not the case in these instances. I think I still have many
of these I need to clean up, but didn't yet get a chance to hit
them all. :(
* Fixed a bunch of busted images.
* Added some missing roles, particularly on landmarks.

There's still more to do - there are several instances in my
old Java/Hibernate examples where I present tabular data in
images (UGH!), that I need to convert to actual tables (why
oh why did I think this was a good idea, even ten years ago?
Live and learn). I've gotta find the blank alt tags. But it's
a start - I can't decide accessibility is going to be a cause
that I'm interested in and not practice it on my own blog.

Everybody resolves to do more blogging all the time, myself
included. Hopefully this gives me a reason to get back
into it!

