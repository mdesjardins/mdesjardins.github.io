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

  I keep reusing a data model for a Pizza shop in my posts, and this post will be no different. This data model first appeared in my <a href="/2008/01/31/new-jpa-tutorial-pizza-shop/">JPA mapping tutorial</a>. Here&#8217;s an ERD of the model again:

<img src="/assets/images/pizza-erd-737223.jpg" border="0" alt="Pizza shop data model ERD" />

<h3>Find me Orders with Small Pizzas!</h3>
Given this model, what if we needed to find each order that contained a small pizza? Suppose your database had the following data:

<table class="table">
  <thead>
    <tr>
      <th>Order Number</th>
      <th>Contents</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>1</td>
      <td>One small, One Large</td>
    </tr>
    <tr>
      <td>2</td>
      <td>Two Smalls, One Large</td>
    </tr>
    <tr>
      <td>3</td>
      <td>Two Large</td>
    </tr>
  </tbody>
</table>

As with [my earlier posting][1], the object model has a PizzaOrder class that contains a Set of Pizza objects which correspond to each customer order. Your first inclination might be to do a criteria-within-a-criteria, like this:

```java
Criteria criteria = Criteria.forClass(PizzaOrder.class);
criteria.createCriteria("pizza").add("pizza_size_id",1);
List
 ordersWithOneSmallPizza = criteria.list();
```

You&#8217;d be in for a bit of a surprise, though. While you might expect only two Pizza orders to be returned (namely, orders #1 and #2), you&#8217;ll actually have three orders in the result set; because order #2 has two small pizzas in it, order #2 will appear twice in your results!

The reason why this happens is pretty simple, and it becomes clear if you enable Hibernate&#8217;s SQL output feature. To locate all of the pizza orders which contain a small pizza, Hibernate needs to do an inner join to the PIZZA table. This is true regardless of whether you&#8217;ve mapped the Pizza objects to be fetched lazily; the join is required because of your query criteria, not because of your mappings. <span style="font-style: italic;">Note: it&#8217;d be really nice if Hibernate were clever enough to identify from the result set that it had duplicate PIZZA_ORDER records, and build the Set of Pizza objects accordingly, but I suspect that this would be a very difficult thing to do, so I&#8217;m not holding my breath.</span>

<span style="font-weight: bold;">The Right Way to Do It</span>  
What you&#8217;re really trying to do is to obtain all Pizza Orders where an associated small pizza exists. In other words, the SQL query that you&#8217;re trying to emulate is

```sql
SELECT *
  FROM PIZZA_ORDER
 WHERE EXISTS (SELECT 1
                 FROM PIZZA
                WHERE PIZZA.pizza_size_id = 1
                  AND PIZZA.pizza_order_id = PIZZA_ORDER.pizza_order_id)
```

The way that you do that is by using an &#8220;exists&#8221; Subquery, like this:

```java
Criteria criteria = Criteria.forClass(PizzaOrder.class,"pizzaOrder");
DetachedCriteria sizeCriteria = DetachedCriteria.forClass(Pizza.class,"pizza");
sizeCriteria.add("pizza_size_id",1);
sizeCriteria.add(Property.forName("pizza.pizza_order_id").eqProperty("pizzaOrder.pizza_order_id"));
criteria.add(Subqueries.exists(sizeCriteria.setProjection(Projections.property("pizza.id"))));
List<pizzaOrder> ordersWithOneSmallPizza = criteria.list();
```

And <span style="font-style: italic;">voila</span>, the result will contain two PizzaOrders!  
<span style="font-style: italic;">Photo Credit: </span>[<span style="font-style: italic;">Squeaky Marmot</span>][2]

 [1]: http://mikedesjardins.net/blog/2008/01/new-jpa-tutorial-pizza-shop.html
 [2]: http://flickr.com/people/squeakymarmot/