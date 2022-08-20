---
title: Configure Spring to automatically re-connect to your EJBs
author: mdesjardins
categories:
  - blog
tags:
  - ejb
  - java
  - spring
---
If you have a service that is a client of a remote EJB, you may have run into the situation where the EJB server shuts down and restarts. When this happens your EJB client may need to be restarted as well, in order to re-discover and reconnect to the EJBs; otherwise you&#8217;ll end up with connection exceptions in the client.

If you&#8217;re using Spring to autowire your EJB clients, it&#8217;s quite easy to configure the service so that the home interface will refresh on connection failures. Note that if you&#8217;re using EJB3, you will need to upgrade to at least version 2.5.5 of Spring. There is a [bug][1] in earlier versions of Spring which prevented this technique from working with EJB3.

In your spring file, make sure you configure your slsb references to have cache-home disabled, and refresh-home-0n-connect-failure thusly:

``` xml
<jee:remote-slsb id="myService" jndi-name="MyService/remote"
        business-interface="us.mikedesjardins.services.MyService"
        cache-home="false" lookup-home-on-startup="false"
        home-interface="us.mikedesjardins.services.MyService"
        resource-ref="false" refresh-home-on-connect-failure="true">
    <jee:environment>
           <!-- Include any relevant environment settings here -->
    </jee:environment>
</jee:remote-slsb>
```

With this, you should be able to restart your EJB hosts without needing to restart your EJB clients!

 [1]: http://jira.springframework.org/browse/SPR-4801
