---
title: RESTful services on FTP?
author: mdesjardins
layout: post
permalink: /2007/04/24/restful-services-on-ftp/
blogger_blog:
  - mikedesjardins.blogspot.com
blogger_author:
  - Mike Desjardinshttp://www.blogger.com/profile/07892630576680251412noreply@blogger.com
blogger_permalink:
  - /2007/04/restful-services-on-ftp.html
categories:
  - blog
tags:
  - REST
---
I&#8217;ve been reading and thinking a lot about REST lately. There&#8217;s a good high-level intro to REST [here][1], [here][2], and (of course) [here][3]. I suppose part of my interest in it stems from my recent projects at work. I was part of a team that exposed a bunch of our product&#8217;s functionality as a brand-new SOAP API. What struck me was how complicated it all is. It&#8217;s <span style="font-style: italic;">supposed</span> to be a simple way to use standardized formats to enable an RPCs across and among disparate technologies, and for the most part, it delivers (Idiosyncrasies like getting a .Net SOAP client to pass dates to a Java SOAP servers, and vice versa, in different timezones, with null dates, [notwithstanding][4]). But the steps involved in creating the service, exposing it, generating the WSDL, embedding that WSDL in the deployed war file, then getting the clients to generate stubs from those WSDLs and making it all work&#8230; well, it&#8217;s cumbersome. More cumbersome than it oughtta be. Full disclosure &#8211; my only exposure to creating SOAP web services is through [Apache Axis][5]. I&#8217;ve heard that CodeHaus&#8217;s [XFire][6] and even .Net are easier to work with.

So reading about RESTful services was like a breath of fresh air. Now, every article I&#8217;ve ever read says that REST does not necessarily need to use HTTP as its transport protocol, but then they invariably proceed to discuss how HTTP&#8217;s GET, POST, PUT, and DELETE commands map really nicely to the ubiquitous CRUD database operations, and explain how to implement REST on HTTP. But I wonder if it wouldn&#8217;t make just as much sense to implement a RESTful service on FTP. Why not? The &#8220;resources&#8221; in both cases are basically file pointers. You could do get, put, and delete, and navigate through the hierarchy of resources with cwd. For transferring large files, the FTP protocol might actually be <span style="font-style: italic;">better</span> suited for the task. So, e.g., let&#8217;s say you have a DSS application where you&#8217;re transferring large batches of data to a data warehouse from an OLTP database. Why wouldn&#8217;t this be a perfect situation for creating a RESTful service over FTP?

 [1]: http://www.xfront.com/REST-Web-Services.html
 [2]: http://webservices.xml.com/pub/a/ws/2002/02/06/rest.html
 [3]: http://en.wikipedia.org/wiki/Representational_State_Transfer
 [4]: http://www.thescripts.com/forum/thread429453.html
 [5]: http://ws.apache.org/axis/
 [6]: http://xfire.codehaus.org/