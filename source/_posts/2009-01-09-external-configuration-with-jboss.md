---
title: External Configuration with JBoss
author: mdesjardins
layout: post
permalink: /2009/01/09/external-configuration-with-jboss/
categories:
  - blog
tags:
  - java
  - jee
  - wireless
---
<img class="alignright size-medium wp-image-213" title="private-property" src="/assets/uploads/2009/01/private-property-300x225.jpg" alt="Private property sign" width="300" height="225" />

In a project I&#8217;m currently working on, I need to make some parameters configurable, and they need to be outside the .war file that I&#8217;m deploying. For example, let&#8217;s say I&#8217;m creating a service which reads data from some other RESTful service. And let&#8217;s say that the other RESTful service has two URLs, one for test and one for production. I&#8217;d like to be able to deploy my .war, and then edit a file outside of that .war file to configure which URL my service should be using.

My first inclination was to try to do this with a JNDI Environment Entry, so I began researching that approach. However, while the EJB spec states in 20.2.4 that the container must &#8220;provide a deployment tool that allows the Deployer to set and modify the values of the enterprise bean&#8217;s environment entries&#8221; (thanks for finding that one, [ipage][1]), JBoss does not seem to have such a facility.

Soon I started to wonder if JNDI wasn&#8217;t a bit overkill for what I needed to do, anyway.  I didn&#8217;t want to specify my parameters on the command-line; I wanted to simplify deployment and wanted to be able to change these values at runtime without restarting the server. But perhaps a System Property was all I needed.

As it turns out, JBoss has the [System Properties Management Service][2] for such things.  Here&#8217;s what you need to do:

1.  Make sure properties-plugin.jar is in your ${JBOSS_HOME}/server/<server>/lib directory.
2.  Make sure the properties-service.xml is in your deploy directory (you can find a copy in the &#8220;default&#8221; server directory)
3.  You now have two options, either edit the URLList to have a comma-separated list of locations of properties files, or you can specify your properties directly in properties-service.xml in the <attribute name=&#8221;Properties&#8221;> element.

Now, to access your property, all you need to do is call the venerable System.getProperty() method.

*Photo Credit: [Shelley Gibb][3]*

 [1]: http://www.jboss.com/?module=bb&op=viewtopic&t=58190
 [2]: http://docs.jboss.org/jbossas/jboss4guide/r3/html/ch10.html
 [3]: http://flickr.com/people/shelleygibb/