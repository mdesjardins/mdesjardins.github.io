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

<div class="wp_syntax">
  <div class="code">
    <pre class="python" style="font-family:monospace;">cs = cairo.<span style="color: black;">ImageSurface</span>.<span style="color: black;">create_from_png</span><span style="color: black;">&#40;</span>iconName<span style="color: black;">&#41;</span>
ct = cairo.<span style="color: black;">Context</span><span style="color: black;">&#40;</span>cs<span style="color: black;">&#41;</span>
ct.<span style="color: black;">set_source_surface</span><span style="color: black;">&#40;</span>cs<span style="color: black;">&#41;</span>
ct.<span style="color: black;">paint</span><span style="color: black;">&#40;</span><span style="color: black;">&#41;</span></pre>
  </div>
</div>

&#8230;and you want to get a Pixbuf from that image. The following function takes the surface, writes it to a PNG that is stored in a string, then uses the Pixbuf loader to load it from that PNG string:

<div class="wp_syntax">
  <div class="code">
    <pre class="python" style="font-family:monospace;"><span style="color: #808080; font-style: italic;"># Stolen from diogodivision's "BlingSwitcher"</span>
<span style="color: #ff7700;font-weight:bold;">def</span> get_pixbuf_from_surface<span style="color: black;">&#40;</span><span style="color: #008000;">self</span>, surface<span style="color: black;">&#41;</span>:
  sio = <span style="color: #dc143c;">StringIO</span><span style="color: black;">&#40;</span><span style="color: black;">&#41;</span>
  surface.<span style="color: black;">write_to_png</span><span style="color: black;">&#40;</span>sio<span style="color: black;">&#41;</span>
  sio.<span style="color: black;">seek</span><span style="color: black;">&#40;</span><span style="color: #ff4500;"></span><span style="color: black;">&#41;</span>
  loader = gtk.<span style="color: black;">gdk</span>.<span style="color: black;">PixbufLoader</span><span style="color: black;">&#40;</span><span style="color: black;">&#41;</span>
  loader.<span style="color: black;">write</span><span style="color: black;">&#40;</span>sio.<span style="color: black;">getvalue</span><span style="color: black;">&#40;</span><span style="color: black;">&#41;</span><span style="color: black;">&#41;</span>
  loader.<span style="color: black;">close</span><span style="color: black;">&#40;</span><span style="color: black;">&#41;</span>
  <span style="color: #ff7700;font-weight:bold;">return</span> loader.<span style="color: black;">get_pixbuf</span><span style="color: black;">&#40;</span><span style="color: black;">&#41;</span></pre>
  </div>
</div>

Neat, huh? This is how I manage to display the temperature on the weather applet. I load up the icon into an ImageSurface, write the text on top of it by calling show\_text, call the function above to convert the ImageSurface to a PixBuf, then call set\_temp_icon using the new PixBuf.

You can see it in action by [downloading the weather applet][2], and looking in weather.py. The relevant code is in the draw\_current\_conditions function.

 [1]: http://cairographics.org
 [2]: http://www.mikedesjardins.us/awn/weather-applet-0.8.tar.gz