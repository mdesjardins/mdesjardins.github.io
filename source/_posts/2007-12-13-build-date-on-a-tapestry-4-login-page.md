---
title: Build Date on a Tapestry 4 login page using Ant
author: mdesjardins
layout: post
permalink: /2007/12/13/build-date-on-a-tapestry-4-login-page/
blogger_blog:
  - mikedesjardins.blogspot.com
blogger_permalink:
  - /2007_12_01_archive.html
categories:
  - blog
tags:
  - ant
  - java
  - tapestry
---
Recently, I had to put a build timestamp onto a login page for a web application I&#8217;m developing at work. The web application is written using Tapestry 4.1, but some of the techniques are equally applicable to other frameworks. I thought I&#8217;d share.

First, you need to setup your ant task to grab a timestamp, and put it into your manifest file. You do so using the tstamp task, like this:  
<span style="font-size:85%;"> <span style="font-family:courier new;"><br /></span></span><a onblur="try {parent.deselectBloggerImageGracefully();} catch(e) {}" href="http://www.mikedesjardins.net/uploaded_images/Screenshot-718923.png"><img style="margin: 0px auto 10px; display: block; text-align: center; cursor: pointer;" src="http://www.mikedesjardins.net/uploaded_images/Screenshot-718919.png" alt="" border="0" /></a>The tstamp task is taking the current date and time, formatting it as specified by pattern (just as you&#8217;d specify it in a [SimpleDateFormat][1]) and placing it in the <span style="font-weight: bold;">buildtstamp</span> variable. The manifest task builds a MANIFEST.MF file which ends up in your deployed web application&#8217;s META-INF directory. You&#8217;ll notice that I&#8217;m also putting the name of the user who built the application into the manifest.

Next, we need to read the Manifest from our application. The first screen presented by my Tapestry app is LogOn.java. First, use [HiveMind][2] to inject the ServletContext into my page:

``` java
@InjectObject(“service:tapestry.globals.ServletContext”)
public abstract ServletContext getServletContext();
```

Also, we need to create an abstract method into which we&#8217;ll store and retrieve the build date:

``` java
public abstract String getBuiltOn();
public abstract void setBuiltOn(String builtOn);
```

Finally, we need to read the Manifest file in our <span style="font-weight: bold;">pageBeginRender</span> method, and set the &#8220;Built On&#8221; date accordingly. This is how I did this: 

``` java
public void pageBeginRender(PageEvent event) {
  ServletContext sc = this.getServletContext();
  String filename = sc.getRealPath("/META-INF/MANIFEST.MF");
  try {  
    BufferedInputStream i = new BufferedInputStream(new FileInputStream(filename));
    Manifest m = new Manifest(i);
    Attributes attrib = m.getMainAttributes();
    this.setBuiltOn(attrib.getValue("Build-Date"));
  } catch (Exception e) {  
    log.warn("Unable to read MANIFEST.MF");
  }
}
```

Finally, we need to actually render this on the LogOn page. I did this with a simple Insert component directly on the html page:

``` 
Built: <span jwcid="@Insert" value="ognl:builtOn"></span>
```

And <span style="font-style: italic;">voila</span>! You have a build date on your log page, which can come in handy, e.g., when your QA team doesn&#8217;t know which version they&#8217;re testing!

 [1]: http://java.sun.com/javase/6/docs/api/java/text/SimpleDateFormat.html
 [2]: http://hivemind.apache.org/