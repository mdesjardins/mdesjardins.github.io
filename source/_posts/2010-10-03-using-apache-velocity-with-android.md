---
title: Using Apache Velocity with Android
author: admin
layout: post
permalink: /2010/10/03/using-apache-velocity-with-android/
categories:
  - blog
tags:
  - android
  - blog
  - java
  - velocity
---
I'm working on an Android project right now where I plan on using a WebView to 
display some content, and I need to generate that content dynamically based on 
the results of a web service request. I wanted an easy-to-use templating 
language to build the pages with. I've worked with both 
<a href="http://velocity.apache.org/" target="_blank">Velocity</a> 
and <a href="http://freemarker.sourceforge.net/" target="_blank">Freemarker</a>, 
and either would've been fine. I settled on Velocity because it was a bit easier 
to set up to work with Android. Here's how I did it.

# Setup Logging

First, I wanted to setup Velocity to use Android's built-in logging system. To do that, 
I needed to create my own logging class that implemented the LogChute interface.

``` java
package com.cereslogic.velocity;
 
import org.apache.velocity.runtime.RuntimeServices;
import org.apache.velocity.runtime.log.LogChute;
 
import android.util.Log;
 
public class VelocityLogger implements LogChute {
    private final static String tag = "Velocity";
     
    @Override
    public void init(RuntimeServices arg0) throws Exception {
    }
 
    @Override
    public boolean isLevelEnabled(int level) {
        return level &gt; LogChute.DEBUG_ID;
    }
 
    @Override
    public void log(int level, String msg) {
        switch(level) {
        case LogChute.DEBUG_ID:
            Log.d(tag,msg);
            break;
        case LogChute.ERROR_ID:
            Log.e(tag,msg);
            break;
        case LogChute.INFO_ID:
            Log.i(tag,msg);
            break;
        case LogChute.TRACE_ID:
            Log.d(tag,msg);
            break;
        case LogChute.WARN_ID:
            Log.w(tag,msg); 
       }
    }
     
    @Override
    public void log(int level, String msg, Throwable t) {
        switch(level) {
        case LogChute.DEBUG_ID:
            Log.d(tag,msg,t);
            break;
        case LogChute.ERROR_ID:
            Log.e(tag,msg,t);
            break;
        case LogChute.INFO_ID:
            Log.i(tag,msg,t);
            break;
        case LogChute.TRACE_ID:
            Log.d(tag,msg,t);
            break;
        case LogChute.WARN_ID:
            Log.w(tag,msg,t);
        }
    }

}
```

You can obviously adjust the **isLevelEnabled** method for your desired logging level.

# Create a ResourceLoader

Next I need to feed my templates to Velocity. I could have read my templates 
manually as files from the assets directory, then passed the contents of the templates 
file to Velocity.evaluate as a String. But Velocity has a very configurable way to 
process templates that, enables it to cache templates internally, so I decided to try that.

When passing Velocity the name of a template file, it delegates the template loading 
to a ResourceLoader class. When you initialize Velocity, you can configure which 
ResourceLoaders it should use to find and read your templates.  Later. when you call 
the **getTemplate** method of the Velocity helper class, you pass it the name of the 
template that you'd like to load as a String.  Velocity will pass the resource name down 
to its ResourceLoader(s).

I wanted to store my Velocity templates in the raw subdirectory of the res directory in 
the Android project, so I needed to build a ResourceLoader that could do that. I 
decided to extend Velocity's built-in FileResourceLoader as a starting point. Here's 
what I came up with:

``` java
    package com.cereslogic.velocity;
     
    import java.io.InputStream;
     
    import org.apache.commons.collections.ExtendedProperties;
    import org.apache.velocity.runtime.RuntimeServices;
    import org.apache.velocity.runtime.resource.Resource;
    import org.apache.velocity.runtime.resource.loader.FileResourceLoader;
     
    import android.content.res.Resources;
 
    public class AndroidResourceLoader extends FileResourceLoader {
        private Resources resources;
        private String packageName;
 
        public void commonInit(RuntimeServices rs, ExtendedProperties configuration) {
            super.commonInit(rs,configuration);
            this.resources = (Resources)rs.getProperty("android.content.res.Resources");
            this.packageName = (String)rs.getProperty("packageName");
        }
 
        public long getLastModified(Resource resource) {
            return ;
        }
 
        public InputStream getResourceStream(String templateName) {
            int id = resources.getIdentifier(templateName, "raw", this.packageName);
            return resources.openRawResource(id);
        }
 
        public boolean  isSourceModified(Resource resource) {
            return false;
        }
     
        public boolean  resourceExists(String templateName) {
            return resources.getIdentifier(templateName, "raw", this.packageName) != ;
        }
    }
```

Because the templates are statically bundled with the .apk file, we can assume that 
Velocity's caches don't need to concern themselves with modification times on the 
templates, which is why **getLastModified** and **isSourceModified** don't really do 
anything.  The **getResourceStream** and **resourceExists** methods lookup the resource 
ID by name. The **commonInit** method is called when the ResourceManager initializes 
the ResourceLoader. You&#8217;ll notice that this is where we stash the package name 
for the resources as well as an instance of the Resource class.

## Use It

So to use what we just created, we need to do some configuration before we call Velocity.init(), which will look something like this:

```
    public class MyActivity extends Activity {
      private void setupVelocity() throws Exception {
            Velocity.setProperty(Velocity.RUNTIME_LOG_LOGSYSTEM_CLASS, "com.cereslogic.velocity.VelocityLogger");
            Velocity.setProperty("resource.loader", "android");
            Velocity.setProperty("android.resource.loader.class", "com.cereslogic.velocity.AndroidResourceLoader");
            Velocity.setProperty("android.content.res.Resources",getResources());
            Velocity.setProperty("packageName", "com.cereslogic.myapplication");
            Velocity.init();
      }
    .
    .
    .
    //
    // Somewhere where we want to use velocity:
    //
        WebView engine = (WebView) findViewById(R.id.web_engine);
        Template template = null;
        try {
            setupVelocity();
            VelocityContext context = new VelocityContext();
            // add stuff to your context.
            template = Velocity.getTemplate("mytemplate");
            StringWriter sw = new StringWriter();
            template.merge(context, sw);
            engine.loadData(sw.toString(), "text/html", "UTF-8");
        } catch (Exception e) {
            // deal with it.
        }
```

In the **setupVelocity** method, we need to configure Velocity to use our 
new ResourceLoader and Logging classes, and configure the package name for 
our resources, just before calling Velocity **init**. Note that, if you 
name your template mytemplate.vm, you'll only pass mytemplate to the 
Velocity **getTemplate** method. This is because of the idiosyncratic way 
that Android's named resource lookup stuff works.

Now you're ready to use Velocity in your Android project!