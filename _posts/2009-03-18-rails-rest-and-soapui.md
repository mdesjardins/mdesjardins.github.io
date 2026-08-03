---
title: Rails, REST, and soapUI
author: mdesjardins
layout: post
permalink: /2009/03/18/rails-rest-and-soapui/
categories:
  - blog
tags:
  - rails
  - REST
  - soapUI
---
I used to use <a href="http://soapui.org" target="_blank">soapUI</a> back when I was creating a SOAP API for a national wireless carrier.  Thankfully, I&#8217;ve been working primarily with REST APIs since then.  I noticed that soapUI had added REST support to their product, and recently I decided to try it out.

The REST API that I&#8217;m developing is in Rails (this is my first foray into Rails; I&#8217;m usually a Java guy with some occasional dabbling in Django and Grails), so the URLs don&#8217;t follow the traditional query-string format for passing parameters.  E.g., instead of GETting **http://server:port/resource?id=123** to retrieve the resource with ID 123, it&#8217;s **http://server:port/resource/123**.

For some reason (admittedly, I was being pretty dense at the time), it took me a long time to figure out how to do this in soapUI, but if you&#8217;ve stumbled into my blog by Googling for how it&#8217;s done, here&#8217;s the trick: first, set your resource URL to **/resource/{id}** (note that id is in curly braces).  Then, when defining your parameters, instead of leaving the STYLE parameter at its default value of QUERY, set it to TEMPLATE, being careful to name your parameter as it is named in the resource URL (in this case, id).  When soapUI generates the request, it&#8217;ll replace the {id} with the parameter&#8217;s value.