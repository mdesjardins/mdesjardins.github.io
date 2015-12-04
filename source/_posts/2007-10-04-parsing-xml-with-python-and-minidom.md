---
title: Parsing XML with Python and minidom
author: mdesjardins
layout: post
permalink: /2007/10/04/parsing-xml-with-python-and-minidom/
blogger_blog:
  - mikedesjardins.blogspot.com
blogger_author:
  - Mike Desjardinshttp://www.blogger.com/profile/07892630576680251412noreply@blogger.com
blogger_permalink:
  - /2007/10/parsing-xml-with-python-and-minidom.html
categories:
  - blog
tags:
  - awn
  - minidom
  - python
  - xml
---
(Continued from my last post)  
So, the first thing I needed to do when creating my weather applet for Avant Window Navigator was actually parse weather data from a weather source. After messing around with Google&#8217;s weather API for a while, I decided to use [weather.com][1]&#8216;s web service. weather.com has a well-documented, straightforward, predictable XML API. To parse the XML, I chose [minidom][2]. Minidom is a &#8220;Lightweight DOM Implementation.&#8221; Here&#8217;s how it works: Let&#8217;s say you have an XML document that supplies a pizza menu, at some URL. Here&#8217;s the XML:

<center>
<img src="/assets/images/pizza-xml-2-706697.png" alt="" border="0" />
</center>

In the python script that will be parsing this, you&#8217;d want to import the minidom package. Let&#8217;s assume that the above XML is served by the URL http://menu.pizzaplace.us, so you&#8217;ll want to import urllib as well. The python code to read up the XML Document might look like the following:

``` python
from xml.dom import minidom
import urllib
import sys
try:
  usock = urllib.urlopen("http://menu.pizzaplace.us")
  xmldoc = minidom.parse(usock)
  usock.close()
except:
  print "Something really bad happened! ", sys.exc_info()[0]
```

Easy, right? Now we want to get the actual data out of the Pizza Menu. Everything in your DOM tree is a Node. This includes text between element tags. In fact, in minidom, the whitespace between strings of text is a node, too (more on that in a minute!). To fetch nodes, you use the <span style="font-weight: bold;">getElementsByTagName</span> function. This function returns a List of nodes with matching element tag names. Another handy function is <span style="font-weight: bold;">getAttribute</span>. As you might expect, it returns the value for an attribute on a particular element. 

Let&#8217;s say we want to iterate through all of the pizzas on the pizza-menu, printing the type of pizza. That code would look like this:

``` python
from xml.dom import minidom
import urllib
try:
  usock = urllib.urlopen("http://menu.pizzaplace.us")
  xmldoc = minidom.parse(usock
  usock.close()
  pizza_list = xmldoc.getElementsByTagName('pizza')
  for pizza_element in pizza_list:
    pizza_type = pizza_element.getAttribute('type')
    print 'Pizza Type: %s' % pizza_type
except:
  print "Something really bad happened! ", sys.exc_info()[0]
```

Next, let&#8217;s pretend that &#8220;heart-attack-special&#8221; pizza sounds really appetizing, and we want to estimate just how much our cholesterol count will spike if we have a slice. We probably want to iterate over the toppings on that pizza to perform that evaluation. To that end, we will hunt for the pizza with the type &#8220;heart-attack-special&#8221;, grab that node, then iterate over the topping sub-nodes. Here&#8217;s how we would do that:

``` python
from xml.dom import minidom
import urllib
try:
  usock = urllib.urlopen("http://menu.pizzaplace.us")
  xmldoc = minidom.parse(usock)
  usock.close()
  pizza_list = xmldoc.getElementsByTagName("pizza")
  for pizza_element in pizza_list:
    pizza_type = pizza_element.getAttribute("type")
    print 'Pizza Type: %s' % pizza_type
    if pizza_type == 'heart-attack-special':
      topping_list = pizza_element.getElementsByName('topping')
      for topping_element in topping_list:
        # (do something here)
except:
  print "Something really bad happened! ", sys.exc_info()[0]
```

As you can see, the pizza_element is a node like any other node, so you can call <span style="font-weight: bold;">getElementsByName</span> on it to get any child nodes of this pizza element. The toppings (pepperoni, sausage, hamburg, canadian bacon, and ham) are themselves child nodes of their respective elements. Each node has a nodeType property which describes the nature of that node. The nodeTypes are TEXT\_NODE, ELEMENT\_NODE, ATTRIBUTE\_NODE, and DOCUMENT\_NODE. Thus, the word &#8220;pepperoni&#8221; is a child node of the first topping node, and is of type TEXT_NODE.

You might be surprised to learn that the fourth topping node on the heart-attack-special is comprised of <span style="font-style: italic;">three</span> child text nodes. The text &#8220;canadian bacon&#8221; has a child with the value bacon, a child with a single character of whitespace, and a child with the value bacon. This is not usually how we want to access the data in our XML documents; we&#8217;d prefer that &#8220;canadian bacon&#8221; be treated as a single node comprised of one string. 

To make the data behave the way we expect it to, we can introduce our own simple utility method called <span style="font-weight: bold;">getText</span>. This function concatenates all child nodes of the supplied node list which are of type TEXT_NODE. It looks like this:

``` python
def getText(nodelist):
  rc = ""
  for node in nodelist:
  if node.nodeType == node.TEXT_NODE:
    rc = rc + node.data
  return rc
```

To use it, we&#8217;d pass it the parent node of the text we&#8217;re interested in. Going back to our original example, we can use the getText function to print out each topping on our heart-attack-special pizza:

``` python
from xml.dom import minidom
import urllib
try:
  usock = urllib.urlopen("http://menu.pizzaplace.us")
  xmldoc = minidom.parse(usock)
  usock.close()
  pizza_list = xmldoc.getElementsByTagName("pizza")
  for pizza_element in pizza_list:
    pizza_type = pizza_element.getAttribute("type")
    print 'Pizza Type: %s' % pizza_type
    if pizza_type == 'heart-attack-special':
      topping_list = pizza_element.getElementsByName('topping')
      for topping_element in topping_list:
        topping_text = getText(topping_element)
        print " Topping %s" % topping_text
except:
  print "Something really bad happened! ", sys.exc_info()[0]
```

The XML-parsing portions of the weather applet that I wrote for the Avant Window Navigator aren&#8217;t much more complicated than this. You can download the source code for the weather applet [here][3]. The parts which parse weather.com&#8217;s data are in the weather.py script, in the <span style="font-weight: bold;">get_conditions</span> and <span style="font-weight: bold;">get_forecast</span> functions.

 [1]: http://xoap.weather.com/
 [2]: http://docs.python.org/lib/module-xml.dom.minidom.html
 [3]: http://www.dragonflymarsh.com/awn/weather-applet-08.tar.gz