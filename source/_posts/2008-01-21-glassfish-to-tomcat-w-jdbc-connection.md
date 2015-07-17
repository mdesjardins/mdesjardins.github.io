---
title: Glassfish to Tomcat w/ JDBC Connection Pooling
author: mdesjardins
layout: post
permalink: /2008/01/21/glassfish-to-tomcat-w-jdbc-connection/
blogger_blog:
  - mikedesjardins.blogspot.com
blogger_author:
  - Mike Desjardinshttp://www.blogger.com/profile/07892630576680251412noreply@blogger.com
blogger_permalink:
  - /2008/01/glassfish-to-tomcat-w-jdbc-connection.html
categories:
  - blog
tags:
  - glassfish
  - hibernate
  - java
  - programming
---
Here&#8217;s your don&#8217;t-waste-hours-on-something-silly-that-I-did tip for the day.

I recently had the opportunity to change an existing in-house web application from running under Tomcat 5.5, to Glassfish v2. The application uses JDBC connection pooling in the application server, and uses JNDI to find the pool. The only stumbling block I encountered was that, immediately after deploying the application, Hibernate would always complain that it couldn&#8217;t find the connection:

```
org.hibernate.HibernateException: Could not find datasource
at org.hibernate.connection.DatasourceConnectionProvider.configure(DatasourceConnectionProvider.java:56)
at org.hibernate.connection.ConnectionProviderFactory.newConnectionProvider(ConnectionProviderFactory.java:124)
at 
.
.
(yadda yadda)
.
Caused by: javax.naming.NameNotFoundException: No object bound to name java:/comp/env/jdbc/MyDB
.
.
```

As it turns out, the datasource that I was using in hibernate.cfg.xml was perfectly fine for Tomcat, but not for Glassfish. I had put the fully qualified JNDI file in my config file:

``` xml
<property name="connection.datasource">java:/comp/env/jdbc/ApptrackDB</property>
```

Glassfish doesn&#8217;t like that. Once I changed it to:

``` xml
<property name="connection.datasource">jdbc/ApptrackDB</property>
```

Everything worked swimmingly.