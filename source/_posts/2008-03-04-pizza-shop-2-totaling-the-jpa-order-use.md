---
title: 'Pizza shop 2: Totaling the JPA Order, use P6Spy to prevent stupidity'
author: mdesjardins
layout: post
permalink: /2008/03/04/pizza-shop-2-totaling-the-jpa-order-use/
blogger_blog:
  - mikedesjardins.blogspot.com
blogger_permalink:
  - /2008_03_01_archive.html
categories:
  - blog
tags:
  - hibernate
  - java
  - jpa
  - p6spy
  - pizzashop
---
I&#8217;m digging up the original Pizza Shop project to illustrate another Hibernate gotcha (it&#8217;s probably applicable to other ORM libraries as well&#8230; admittedly, I haven&#8217;t tested it). If you didn&#8217;t read the original Pizza Shop post, you can find it [here][1]. To review, here is an ERD for the system:  
<center>
<img alt="" src="/assets/images/pizza-erd-737227.jpg" border="0" />
</center>

Today I&#8217;d like to show how <i>not</i> to total up the cost of the customer order. First, we are going to add [P6Spy][2] to the project. P6Spy is an excellent &#8220;JDBC wrapper&#8221; tool that sits between your application code and your actual JDBC driver. It intercepts your application&#8217;s JDBC requests and logs the results. It&#8217;s an invaluable tool for optimizing the voodoo out of an ORM tool, and the great thing is that it&#8217;s simple to setup:

1.) Change your application&#8217;s JDBC driver from whatever it currently is (e.g., org.postgresql.Driver), to the P6Spy JDBC driver, com.p6spy.engine.spy.P6SpyDriver.  
2.) Modify the spy.properties file by editing the line that starts with &#8220;realdriver=&#8221;, changing the value to your actual JDBC driver.  
3.) Put spy.properties on your classpath.

That&#8217;s it! Now that we have that out of the way, let&#8217;s see just how badly we can screw up a simple method in our Model class. The method we are going to add will total up the price of an order. If you look at the ERD above, you can infer that the total cost of an order is the sum of the base price for each pizza, depending on its size, plus the price of each pizza&#8217;s individual toppings.

So for our example order, let&#8217;s say we have the following:  
1 Small Pizza with Pepperoni and Mushroom  
1 Medium Pizza with Sausage and Onions  
1 Large Pizza with Extra Cheese

You&#8217;ll recall that we created a Model class which provides a method for retrieving a List of Pizza objects that are associated with an order ID. The temptation is to create a method which looks something like this:

``` java
public BigDecimal getOrderPriceWrong(Integer orderId) {
  BigDecimal result = new BigDecimal(0.0);
  Order order = this.getOrder(orderId);
  for (Pizza pizza : order.getPizzas()) {
    result.add(pizza.getSize().getBasePrice());
    for (Topping topping : pizza.getToppings()) {
      result.add(topping.getPrice());
    }
  }
  return result;
}
```

Because Hibernate does lazy-fetching, it&#8217;s not going to attempt to calculate the total cost with as few queries as possible. Instead, Hibernate&#8217;s general philosophy is to defer any queries until it knows that it absolutely needs to do them, substituting empty proxy objects for populated ones until required. Usually this is an optimization, but in this case, Hibernate will do the following queries:

1.) Query to obtain an order object.  
2.) One Query for each Pizza, joined to the Size, to obtain the base price.  
3.) One Query per Topping, to obtain the topping price.

In all, we get nine individual SQL queries to compute the total price of our single fictional order. The proof is in the P6Spy&#8217;s output, spy.log, truncated below:

    select order0_.pizza_order_id as pizza1_2_, order0_.version ...
    select pizzas0_.pizza_order_id as pizza4_2_, pizzas0_.pizza_id ...
    select pizzas0_.pizza_order_id as pizza4_2_, pizzas0_.pizza_id ...
    select pizzas0_.pizza_order_id as pizza4_2_, pizzas0_.pizza_id ...
    select toppings0_.pizza_id as pizza1_1_, toppings0_.topping_id ...
    select toppings0_.pizza_id as pizza1_1_, toppings0_.topping_id ...
    select toppings0_.pizza_id as pizza1_1_, toppings0_.topping_id ...
    select toppings0_.pizza_id as pizza1_1_, toppings0_.topping_id ...
    select toppings0_.pizza_id as pizza1_1_, toppings0_.topping_id ...

If your application is totaling up the price of Pizza orders all day, this can really add up! An alternative approach is to use two named queries to compute the total base price, and total topping price, for an order. For example, we might add the following annotation to the Pizza class:

``` java
@NamedQueries(
{
@NamedQuery(
 name="basePrice",
 query="select SUM(p.size.basePrice) " +
       "  from Pizza p " +
       " where p.order.id = :orderId"),
@NamedQuery(
 name="toppingPrice",
 query="select SUM(topping.price) " +
       "  from Pizza p join p.toppings as topping " +
       " where p.order.id = :orderId")
})
```

Then, you could add the following methods to the OrderDao:

``` java
public BigDecimal nullGuard(Query query) {
  BigDecimal result = (BigDecimal)query.getSingleResult();
  return (result == null ? new BigDecimal() : result);
}
 
public BigDecimal getOrderPrice(Integer orderId) {
  Query query1 = getEntityManager().createNamedQuery("basePrice");
  query1.setParameter("orderId",orderId);
  Query query2 = getEntityManager().createNamedQuery("toppingPrice");
  query2.setParameter("orderId", orderId);
  return nullGuard(query1).add(nullGuard(query2));
}
```

&#8230;and call into the DAO from the Model class, thusly:

``` java
public BigDecimal getOrderPriceRight(Integer orderId) {
  return this.orderDao.getOrderPrice(orderId);
}
```

When we run the P6Spy test now, we see a meager two queries where we used to have nine:

    select SUM(size1_.pizza_size_base_price) as col_0_0_ from PIZZA ...
    select SUM(topping2_.topping_price) as col_0_0_ from PIZZA ...

It pays to periodically use a tool like P6Spy on your application, to look for easy wins like this one!

I&#8217;ve included a complete working eclipse project that demonstrates this&#8230; it&#8217;s actually a tweaked version of the earlier Pizza Shop project. You can get it [here][3].

 [1]: http://blog.mikedesjardins.us/2008/01/new-jpa-tutorial-pizza-shop.html
 [2]: http://www.p6spy.com/
 [3]: http://www.mikedesjardins.us/pizzashop-2.0.tar.gz