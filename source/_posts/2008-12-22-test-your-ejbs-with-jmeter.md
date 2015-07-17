---
title: Test your EJBs with JMeter
author: mdesjardins
layout: post
permalink: /2008/12/22/test-your-ejbs-with-jmeter/
categories:
  - blog
tags:
  - ejb
  - java
  - jmeter
  - programming
---
<img class="alignright size-thumbnail wp-image-198" title="jaguar-tachometer.jpg" src="http://www.cereslogic.com/pages/wp-content/uploads/2008/12/jaguar-tachometer-150x150.jpg" alt="jaguar-tachometer.jpg" width="150" height="150" />

Sometimes it&#8217;s helpful to do some performance benchmarks on your EJBs. There are a few different ways to do this, but I&#8217;ve found that Apache&#8217;s [JMeter][1] is an excellent tool for benchmarking. Unfortunately, JMeter doesn&#8217;t come with a general-purpose sampler for testing arbitrary EJBs. Luckily, it isn&#8217;t very difficult to create one.

For this article, I&#8217;m using the JBoss application server to host my EJBs. The process for using other containers should be quite similar.

### 1.) Create a factory to lookup your EJBs.

The first thing that you&#8217;ll probably want to do is create a simple singleton factory class to create instances of your EJB client for your test. I use something like the following:


```java
public class MyServiceFactory {
  private static final Log log = LogFactory.getLog(MyServiceFactory.class);
  private static MyService service;
  private static MyServiceFactory me;
 
  private MyServiceFactory() { }
 
  static {
    MyServiceFactory.me = new MyServiceFactory();
  }
 
  public static MyServiceFactory getInstance() {
    return MyServiceFactory.me;
  }
 
  public MyService getService() {
    if (MyService.service == null) {
      // Get the remote interface of the music search service
      try {
        log.info("Loading the service...");
 
        // JNDI the old-fashioned way:
        Context ctx = new InitialContext();
        service = (MyService)ctx.lookup("MyAction/remote");
        if (service == null) {
          log.error("Didn't get the service!");
        }
      } catch (NamingException e) {
        log.error("Error looking up the remote service", e);
        return null;
      }
    }
    return service;
  }
}
```


### 2.) Write the test

Next, we&#8217;ll need to write the test itself. To do this, we&#8217;ll extend the AbstractJavaSamplerClient class in JMeter&#8217;s org.apache.jmeter.protocol.java.sampler package. This abstract class has a runTest method that we will override, and this method implements the actual test. We will also override the getDefaultParameters method to provide some reasonable defaults values which will be displayed in JMeter&#8217;s GUI application.

``` java
package us.mikedesjardins.demo.jmeter;
 
import org.apache.jmeter.config.Arguments;
import org.apache.jmeter.protocol.java.sampler.AbstractJavaSamplerClient;
import org.apache.jmeter.protocol.java.sampler.JavaSamplerContext;
import org.apache.jmeter.samplers.SampleResult;
 
public class DigitalContentServiceEJBTestSampler extends AbstractJavaSamplerClient {
  public SampleResult runTest(JavaSamplerContext context) {
    SampleResult results = new SampleResult();
    MyService service = MyServiceFactory.getInstance().getService();
 
    results.sampleStart();
    Long param1 = context.getLongParameter("PARAM_1");
    String param2 = context.getStringParameter("PARAM_2");
 
    MyResult result = service.myMethod(param1, param2);
    if (result != null) {
       results.setSuccessful(true);
       results.setResponseCodeOK();
       results.setResponseMessage("'myResult:" + myResult);
    } else {
       results.setSuccessful(false);
    }
    results.sampleEnd();
    return results;
  }
 
  @Override
  public Arguments getDefaultParameters() {
    Arguments args = new Arguments();
    args.addArgument("PARAM_1", "4815162342");
    args.addArgument("PARAM_2", "Iculus");
    return args;
  }
}
```

### 3.) Setup JMeter

JMeter&#8217;s extra libs directory is ${JMETER\_INSTALL\_LIB}/lib/ext. Into that directory you will need to copy any jars that your EJB client will need. In you&#8217;re using JBoss, you will want to copy the jbossall-client.jar into that directory as well (for the JNDI client and other remoting goodies) &#8211; presumably other application servers have similar client jar files available.

When you fire up JMeter, your new sampler should appear in the Samplers menu. Enjoy!

*Photo Credit: [Bill Jacobus][2]*

 [1]: http://jakarta.apache.org/jmeter
 [2]: http://flickr.com/people/billjacobus1/