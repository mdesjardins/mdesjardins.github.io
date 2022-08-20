---
title: Python and GConf
author: mdesjardins
layout: post
permalink: /2007/11/16/python-and-gconf/
blogger_blog:
  - mikedesjardins.blogspot.com
blogger_author:
  - Mike Desjardinshttp://www.blogger.com/profile/07892630576680251412noreply@blogger.com
blogger_permalink:
  - /2007/11/python-and-gconf.html
categories:
  - blog
tags:
  - awn
  - gconf
  - python
---
The [weather applet][1] and [clock/calendar applet][2] for the [Avant Window Navigator][3] both use [GConf][4] to store their configuration settings. GConf is part of the GNOME environment on Linux. It maintains a hierarchical set of configuration data in (key,value) pairs, much like the registry on Windows or the &#8220;plist files&#8221; on OSX. One of the nice things about GConf is that you can register your application to receive notifications in a callback function whenever any interesting configuration values change.

I noticed that there weren&#8217;t a lot of tutorials out there on the topic of mixing Python with GConf, so I decided to write one.

First, you&#8217;ll need to import the Python bindings for GConf:

``` python
import gconf
```

Next, you want to create a GConf &#8220;client&#8221; in your Python app. You&#8217;ll probably want to do this in an \_\_init\_\_() function somewhere. You&#8217;ll also want to register your application to receive notifications when ever your configuration values change. The hierarchy of configuration values in GConf follows the familiar slash-separated path notation. For example, the configuration values for my AWN weather applet are stored in /apps/avant-window-navigator/applets/weather. So, if I want to register a callback named config_event to be called whenever configuration values on that path are modified, I&#8217;d do the following:

``` python
self.gconf_client = gconf.client_get_default()
self.gconf_client.notify_add("/apps/avant-window-navigator/applets/weather", self.config_event)
```

<span style="font-size:100%;">I usually write a generic &#8220;get_config&#8221; function, and have the callback call get_config. That way, I can use the same configuration code when I initialize. The config callback then looks simple&#8230; something like this:</p> 

``` python
def config_event(self, gconf_client, *args, **kwargs):
  self.get_config()
```


The kwargs parameter gives you a list of parameters that changed. You can fine-tune your configuration code based 
on this, but I usually just ignore it and re-read everything because I don&#8217;t usually have very many parameters.

GConf provides functions for reading your parameters. They look like this:

``` python
foo = self.gconf_client.get_string("/path/to/my/config/data/foo")
bar = self.gconf_client.get_int("/path/to/my/config/data/bar")
baz = self.gconf_client.get_bool("/path/to/my/config/data/baz")
```

All of the functions except </span><span style=";font-family:courier new;font-size:100%;"  >get_bool</span>return None 
if the key isn&#8217;t found. Oddly, <span style=";font-family:courier new;font-size:100%;"  >get_bool</span> seems to 
return False if the key isn&#8217;t found. In my configuration code, I like to initialize my GConf values when the key 
isn&#8217;t found. So when if my code were to read the &#8220;foo&#8221; parameter like the above example, it&#8217;d 
actually be coded something like this:</span>
  
``` python
foo = self.gconf_client.get_string("/path/to/my/config/foo")
if foo == None:
  self.gconf_client.set_string("/path/to/my/config/foo", "Default Value")
  foo = "Default Value"
```

And I usually wrap the above idiom in its own function that accepts a key name and 
a default value.

<center>
<img src="/assets/images/Screenshot-Configuration-Editor---weather-703844.png" alt="" border="0" /></center><br /><span style="font-size:100%;">

Note that you can edit and interact with your GConf settings in realtime using </span><span style="font-size:100%;">the GNOME configuration tool. If you 
use Ubuntu, then this utility may be found under the &#8220;System Tools&#8221; menu. Editing configuration values in the configuration tool will result 
in your callback being executed as you might expect.

This also makes creating a configuration dialog easy. The configuration dialog just needs to write its updated values to gconf when the user clicks Apply or OK. If you&#8217;ve created a callback and generic configuration function, then the application will automatically reconfigure itself after the user applies their modifications!

 [1]: http://wiki.awn-project.org/index.php?title=Weather_Applet
 [2]: http://wiki.awn-project.org/index.php?title=Clock/Calendar_Applet
 [3]: http://www.awn-project.org/
 [4]: http://www.gnome.org/projects/gconf/