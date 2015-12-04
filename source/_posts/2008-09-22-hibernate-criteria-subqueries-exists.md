---
title: 'Hibernate Criteria Subqueries: Exists'
author: mdesjardins
layout: post
permalink: /2008/09/22/hibernate-criteria-subqueries-exists/
blogger_blog:
  - mikedesjardins.blogspot.com
blogger_permalink:
  - /2008_09_01_archive.html
categories:
  - blog
tags:
  - hibernate
  - java
  - jpa
  - pizzashop
---
<center>
<img src="/assets/images/national-pizza-shop-718008.jpg" border="0" alt="" />
</center>

While working on a recent project, I came into a situation where I needed to do an &#8220;exists&#8221; query, using a Criteria-style query. The online documentation for this feature is a little sparse, so I thought I&#8217;d share what I did.

<span style="font-weight: bold;">The Pizza-shop Data Model (Again)</span>

<div style="text-align: left;">
  I keep reusing a data model for a Pizza shop in my posts, and this post will be no different. This data model first appeared in my <a href="http://mikedesjardins.us/blog/2008/01/new-jpa-tutorial-pizza-shop.html">JPA mapping tutorial</a>. Here&#8217;s an ERD of the model again:
</div>

<a href="http://mikedesjardins.net/uploaded_images/pizza-erd-737223.jpg" onblur="try {parent.deselectBloggerImageGracefully();} catch(e) {}"><img src="/assets/images/pizza-erd-737223.jpg" border="0" alt="" /></a><span style="font-weight: bold;">Find me Orders with Small Pizzas!</span>  
Given this model, what if we needed to find each order that contained a small pizza? Suppose your database had the following data:

<a href="http://mikedesjardins.net/uploaded_images/table-752077.jpg" onblur="try {parent.deselectBloggerImageGracefully();} catch(e) {}"><img src="/assets/images/table-752035.jpg" border="0" alt="" /></a>  
As with [my earlier posting][1], the object model has a PizzaOrder class that contains a Set of Pizza objects which correspond to each customer order. Your first inclination might be to do a criteria-within-a-criteria, like this:

<div class="wp_syntax">
  <div class="code">
    <pre class="java" style="font-family:monospace;">Criteria criteria <span style="color: #339933;">=</span> Criteria.<span style="color: #006633;">forClass</span><span style="color: #009900;">&#40;</span>PizzaOrder.<span style="color: #000000; font-weight: bold;">class</span><span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
criteria.<span style="color: #006633;">createCriteria</span><span style="color: #009900;">&#40;</span><span style="color: #0000ff;">"pizza"</span><span style="color: #009900;">&#41;</span>.<span style="color: #006633;">add</span><span style="color: #009900;">&#40;</span><span style="color: #0000ff;">"pizza_size_id"</span>,<span style="color: #cc66cc;">1</span><span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
<span style="color: #003399;">List</span>
 ordersWithOneSmallPizza <span style="color: #339933;">=</span> criteria.<span style="color: #006633;">list</span><span style="color: #009900;">&#40;</span><span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span></pre>
  </div>
</div>

You&#8217;d be in for a bit of a surprise, though. While you might expect only two Pizza orders to be returned (namely, orders #1 and #2), you&#8217;ll actually have three orders in the result set; because order #2 has two small pizzas in it, order #2 will appear twice in your results!

The reason why this happens is pretty simple, and it becomes clear if you enable Hibernate&#8217;s SQL output feature. To locate all of the pizza orders which contain a small pizza, Hibernate needs to do an inner join to the PIZZA table. This is true regardless of whether you&#8217;ve mapped the Pizza objects to be fetched lazily; the join is required because of your query criteria, not because of your mappings. <span style="font-style: italic;">Note: it&#8217;d be really nice if Hibernate were clever enough to identify from the result set that it had duplicate PIZZA_ORDER records, and build the Set of Pizza objects accordingly, but I suspect that this would be a very difficult thing to do, so I&#8217;m not holding my breath.</span>

<span style="font-weight: bold;">The Right Way to Do It</span>  
What you&#8217;re really trying to do is to obtain all Pizza Orders where an associated small pizza exists. In other words, the SQL query that you&#8217;re trying to emulate is

<div class="wp_syntax">
  <div class="code">
    <pre class="sql" style="font-family:monospace;"><span style="color: #993333; font-weight: bold;">SELECT</span> <span style="color: #66cc66;">*</span>
  <span style="color: #993333; font-weight: bold;">FROM</span> PIZZA_ORDER
 <span style="color: #993333; font-weight: bold;">WHERE</span> <span style="color: #993333; font-weight: bold;">EXISTS</span> <span style="color: #66cc66;">&#40;</span><span style="color: #993333; font-weight: bold;">SELECT</span> <span style="color: #cc66cc;">1</span>
                 <span style="color: #993333; font-weight: bold;">FROM</span> PIZZA
                <span style="color: #993333; font-weight: bold;">WHERE</span> PIZZA<span style="color: #66cc66;">.</span>pizza_size_id <span style="color: #66cc66;">=</span> <span style="color: #cc66cc;">1</span>
                  <span style="color: #993333; font-weight: bold;">AND</span> PIZZA<span style="color: #66cc66;">.</span>pizza_order_id <span style="color: #66cc66;">=</span> PIZZA_ORDER<span style="color: #66cc66;">.</span>pizza_order_id<span style="color: #66cc66;">&#41;</span></pre>
  </div>
</div>

The way that you do that is by using an &#8220;exists&#8221; Subquery, like this:

<div class="wp_syntax">
  <div class="code">
    <pre class="java" style="font-family:monospace;">Criteria criteria <span style="color: #339933;">=</span> Criteria.<span style="color: #006633;">forClass</span><span style="color: #009900;">&#40;</span>PizzaOrder.<span style="color: #000000; font-weight: bold;">class</span>,<span style="color: #0000ff;">"pizzaOrder"</span><span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
DetachedCriteria sizeCriteria <span style="color: #339933;">=</span> DetachedCriteria.<span style="color: #006633;">forClass</span><span style="color: #009900;">&#40;</span>Pizza.<span style="color: #000000; font-weight: bold;">class</span>,<span style="color: #0000ff;">"pizza"</span><span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
sizeCriteria.<span style="color: #006633;">add</span><span style="color: #009900;">&#40;</span><span style="color: #0000ff;">"pizza_size_id"</span>,<span style="color: #cc66cc;">1</span><span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
sizeCriteria.<span style="color: #006633;">add</span><span style="color: #009900;">&#40;</span>Property.<span style="color: #006633;">forName</span><span style="color: #009900;">&#40;</span><span style="color: #0000ff;">"pizza.pizza_order_id"</span><span style="color: #009900;">&#41;</span>.<span style="color: #006633;">eqProperty</span><span style="color: #009900;">&#40;</span><span style="color: #0000ff;">"pizzaOrder.pizza_order_id"</span><span style="color: #009900;">&#41;</span><span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
criteria.<span style="color: #006633;">add</span><span style="color: #009900;">&#40;</span>Subqueries.<span style="color: #006633;">exists</span><span style="color: #009900;">&#40;</span>sizeCriteria.<span style="color: #006633;">setProjection</span><span style="color: #009900;">&#40;</span>Projections.<span style="color: #006633;">property</span><span style="color: #009900;">&#40;</span><span style="color: #0000ff;">"pizza.id"</span><span style="color: #009900;">&#41;</span><span style="color: #009900;">&#41;</span><span style="color: #009900;">&#41;</span><span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
List<span style="color: #339933;">&lt;</span>pizzaOrder<span style="color: #339933;">&gt;</span> ordersWithOneSmallPizza <span style="color: #339933;">=</span> criteria.<span style="color: #006633;">list</span><span style="color: #009900;">&#40;</span><span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span></pre>
  </div>
</div>

And <span style="font-style: italic;">voila</span>, the result will contain two PizzaOrders!  
<span style="font-style: italic;">Photo Credit: </span>[<span style="font-style: italic;">Squeaky Marmot</span>][2]

 [1]: http://mikedesjardins.net/blog/2008/01/new-jpa-tutorial-pizza-shop.html
 [2]: http://flickr.com/people/squeakymarmot/