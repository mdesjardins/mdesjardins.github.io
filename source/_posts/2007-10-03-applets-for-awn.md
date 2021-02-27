---
title: Applets for AWN
author: mdesjardins
layout: post
permalink: /2007/10/03/applets-for-awn/
blogger_blog:
  - mikedesjardins.blogspot.com
blogger_permalink:
  - /2007_10_01_archive.html
categories:
  - blog
tags:
  - awn
  - python
---
I use Linux on the desktop at work. As a Mac owner, I was frustrated by the lack of a usable Dock, until I stumbled across a project called the [Avant Window Navigator][1]. Not only is it a cool dock, but it comes with oodles of eye-candy, which I&#8217;m a sucker for.

Here&#8217;s a screenshot of my dock:

<center>
<img src="/assets/images/Screenshot-761480.png" alt="" border="0" />
</center>

You hear a lot about open source developers who got involved because they needed to &#8220;scratch an itch.&#8221; I happen to do web development in Java, and I had come to rely on my Gnome CPU Meter applet to tell me when Tomcat had finished deploying (there was always a spike while it was deploying a war file, and then a dropoff when it was finished). I didn&#8217;t have that in AWN. Fortunately, you can make applets for it.

So the first applet I created, after a &#8220;Hello World&#8221; applet and a clock, was a C-based applet called CPU Meter (incidentally, my &#8220;Hello World&#8221; applet is now part of the awn-extras package). For the CPU Meter, I stole some code from the [Gnome System Monitor][2], mashed it into my Hello World applet, read up on the [Cairo Graphics API][3] and [<span style="text-decoration: underline;">GTK+ API</span>][4], and cobbled together a working CPU load grapher. Later on, another developer came up with an applet called &#8220;AwnTop&#8221; (a process listing applet), and he merged my stuff in with his stuff. The new applet is now called the AWN System Monitor.

Next, I decided to create a weather applet. All of the other developers had started writing their applets in Python, so I figured it was as good time as any to learn Python! I&#8217;ve gotta say I LOVE Python. It&#8217;s JavaScript&#8217;s simplicity, Shell scripting&#8217;s ease-of-use, and Perl&#8217;s power&#8230; and it&#8217;s incredibly easy to read and learn. Here&#8217;s a picture of the weather applet in action:

<center>
<img src="/assets/images/Screenshot-1-779652.png" alt="" border="0" /></center>Cool, huh?

Well, I really didn&#8217;t intend for this post to be a brag session. I wanted it to be a how-to develop applets for AWN using Python. But now I guess I&#8217;ll leave that for the next post!

 [1]: http://wiki.awn-project.org/
 [2]: http://www.blogger.com/img/gl.link.gif
 [3]: http://www.cairographics.org/
 [4]: http://library.gnome.org/devel/gtk/2.12/