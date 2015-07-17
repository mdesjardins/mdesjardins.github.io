---
title: 'Who is running that horrible Hibernate query?  Find out with comments'
author: mdesjardins
layout: post
permalink: /2008/06/01/who-is-running-that-horrible-hibernate/
blogger_blog:
  - mikedesjardins.blogspot.com
blogger_permalink:
  - /2008_06_01_archive.html
categories:
  - blog
tags:
  - hibernate
  - jpa
---
<span style="font-weight: bold;">The Dreaded Search Function</span>  
I&#8217;ve worked at several jobs where the users have asked for a general-purpose search function in their application. It&#8217;s the sort of thing where people want the ability to search on, e.g., all customers with a last name that starts with SM, or all of the customers with a balance greater than 1000.00, etc.

If you can resist the request to build such a thing, <span style="font-style: italic;">then by all means, <span style="font-weight: bold;">don&#8217;t implement it</span></span>. End-users have a knack for creating queries which will bring your database to its knees, no matter how clever you think you are with indexing or UI design.

If you must do it, you&#8217;ll want to keep a close eye on how its used from the outset for when the inevitable nightmare query is run. One way to do this is by using Hibernate&#8217;s comments feature.

<span style="font-weight: bold;">Use Comments to Hunt them Down</span>  
My colleague [Matt Brock][1] implemented something like this where we work. Suppose you have a general method for creating an HQL query in a DAO class, the method could look like this:

``` java
protected Query createQuery(String hql) {
return getSession().createQuery(hql).setComment(getUsername());
}
```

Next, make sure Hibernate&#8217;s SQL logging and comment logging is enabled in your hibernate.hbm.xml (warning &#8211; this log can get pretty huge):

<div class="wp_syntax">
  <div class="code">
    <pre class="xml" style="font-family:monospace;">&nbsp;
true
true
.
.
.</pre>
  </div>
</div>

Your log will now contain the users who are executing each query, like this:

```
Hibernate: / johndoe / select customer0.customer_id as col_0_0, customer0.cust_company_name from CUSTOMER customer0 where (customer0.customer_id=12136 or customer0.customer_id=16884 or customer0.customer_id=11150 or customer0.customer_id=155 or customer0.customer_id
=27265 or customer0.customer_id=697 or customer0.customer_id=4133 or customer0.customer_id=248 or customer0.customer_id=2550 or customer0.customer_id=8449)
```

One caveat: this may not work with stored procedures &#8211; SQL Server, in particular, will get nasty if you try to execute a stored procedure with Hibernate comments attached to it.

Happy Hibernating!

 [1]: http://mattbrock.net/