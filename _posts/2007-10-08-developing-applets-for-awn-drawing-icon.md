---
title: 'Developing Applets for AWN: Drawing the Icon'
author: mdesjardins
layout: post
permalink: /2007/10/08/developing-applets-for-awn-drawing-icon/
blogger_blog:
  - mikedesjardins.blogspot.com
blogger_author:
  - Mike Desjardinshttp://www.blogger.com/profile/07892630576680251412noreply@blogger.com
blogger_permalink:
  - /2007/10/developing-applets-for-awn-drawing-icon.html
categories:
  - blog
tags:
  - awn
  - cairo
  - python
---
(Continued from previous two posts)  
If you&#8217;re familiar with the Avant Window Navigator, then you know that a fair amount of the project revolves around eye candy and visual effects. Making an applet that is consistent with the user&#8217;s expectations with respect to visual effects (e.g., reflection, bouncing, rotating, etc.) is very important. Fortunately, Neil and Co. have made this quite straightforward. By simply sub-classing the awn.AppletSimple class, your applet will inherit all of the special effects that it should. 

Unfortunately, there&#8217;s one small trick you need to work around. If your applet is so simple that it just needs a single, static icon, then sub-classing awn.AppletSimple, followed by a call to set_icon, works great. It gets more difficult if you need to draw dynamic content in the area usually occupied by the icon.

The problem is that set\_icon (as well as its misunderstood cousin, set\_temp_icon) takes a Pixbuf as its input parameter. However, the drawing framework for doing just about anything, especially loading PNGs, is [cairo][1]. Cairo has no native support for converting a surface to a Pixbuf. I discovered a trick for doing this by looking at the source code for the PyClock and BlingSwitcher applets. Let&#8217;s say you have a Cairo image surface that you&#8217;ve created from an existing PNG, thusly:

```python
cs = cairo.ImageSurface.create_from_png(iconName)
ct = cairo.Context(cs)
ct.set_source_surface(cs)
ct.paint()
```

&#8230;and you want to get a Pixbuf from that image. The following function takes the surface, writes it to a PNG that is stored in a string, then uses the Pixbuf loader to load it from that PNG string:

```python
# Stolen from diogodivision's "BlingSwitcher"
def get_pixbuf_from_surface(self, surface):
  sio = StringIO()
  surface.write_to_png(sio)
  sio.seek()
  loader = gtk.gdk.PixbufLoader()
  loader.write(sio.getvalue())
  loader.close()
  return loader.get_pixbuf()
```

Neat, huh? This is how I manage to display the temperature on the weather applet. I load up the icon into an ImageSurface, write the text on top of it by calling show\_text, call the function above to convert the ImageSurface to a PixBuf, then call set\_temp_icon using the new PixBuf.

You can see it in action by [downloading the weather applet][2], and looking in weather.py. The relevant code is in the draw\_current\_conditions function.

 [1]: http://cairographics.org
 [2]: http://www.mikedesjardins.us/awn/weather-applet-0.8.tar.gz