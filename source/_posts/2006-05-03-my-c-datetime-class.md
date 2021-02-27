---
title: My C++ DateTime class
author: mdesjardins
layout: post
permalink: /2006/05/03/my-c-datetime-class/
blogger_blog:
  - mikedesjardins.blogspot.com
blogger_permalink:
  - /2006_05_01_archive.html
categories:
  - blog
tags:
  - c++
  - datetime
---
My C++ Date/Time class is now available on this site under the “Geeky” tab on the navbar. I was hoping to do so much more with it. I wanted documentation (maybe with doxygen), I wanted to compile it on a bunch of different compilers and beef up the makefile, I wanted automated unit tests. In the end I just realized that I’d never have the time to “finish” it, so I did what all of the other open source developers do, and put a half-finished, mostly-useful chunk of code out there. The main difference is that <u>nobody </u>will care about my date class. Probably nobody will even read this.

I wrote it because I didn’t like the date class I was using where I worked, and I really didn’t want to use the one available at Boost.org. I wanted something stupid simple to use, like Java’s java.util.Date, and it’s companion, SimpleDateFormatter. Why can’t C++ be more like Java? There are things that can make C++ more usable which don’t affect the language at all. The biggest thing is the libraries and the packaging. You can’t beat importing a package from a jar file. And you can’t beat having things like HttpClient and Date available in the libraries that come with the language. I know the latter will never happen… the beauty of having one company “control” a language is that decisions can be made about APIs quickly and easily. That will never happen in a standardization committee. 

Oh, well.