---
title: 'Use Hibernate&#8217;s Custom Loaders to fake an aggregation view'
author: mdesjardins
layout: post
permalink: /2008/06/23/use-hibernates-custom-loaders-to-fake/
blogger_blog:
  - mikedesjardins.blogspot.com
blogger_author:
  - Mike Desjardinshttp://www.blogger.com/profile/07892630576680251412noreply@blogger.com
blogger_permalink:
  - /2008/06/use-hibernates-custom-loaders-to-fake.html
categories:
  - blog
tags:
  - hibernate
  - java
  - jpa
  - programming
---
<span style="font-weight: bold;">Your Problem</span>  
You have a data model with table that contains data you want to aggregate. For instance (returning to my venerable Pizza Shop example), let&#8217;s say you have a PRODUCT table that enumerates the items your pizza shops sells, and a LOCATION table that contains all of your retail locations:

<center>
<img src="/assets/_images/blog-location-product-791813.png" alt="" border="0" />
</center>
We also have a table that contains the sum of all of the previous days&#8217; sales, broken down by PRODUCT and LOCATION:

<center>
<img src="/assets/images/blog-yesterday-sales-786525.png" alt="" border="0" />
</center>
This is all well and good, but we&#8217;ve been asked to create an Executive Dashboard for the President of the pizza chain, and she would like to see daily sales by product. She is not interested in a breakdown by location.

<span style="font-weight: bold;">We could tally it up client side&#8230;</span>  
What if we just loaded the entire table into the client, and iterated over the per-product results, and present that? Unfortunately, ORM libraries are pretty stupid in situations like these, and will generate all kinds of expensive reads to the database when you try to solve the problem this way. If you want to learn more about lazy loading, and why you shouldn&#8217;t iterate over a collection, check out my earlier post [here][1].

<span style="font-weight: bold;">We could just create a view in the Database&#8230;</span>  
We <span>could</span><span style="font-weight: bold;"> </span>just create a view, and aggregate the data there. Then we can easily create a Hibernate mapping to that view. The query for the view is simple:

``` sql
CREATE VIEW sales_by_product_view AS
SELECT product_id, SUM(total_sales) AS total_sales
  FROM YESTERDAY_SALES
 GROUP BY product_id
```

However, we are working with a tyrannical DBA. She is not keen on proliferating views throughout our otherwise pristine schema every time the President has decided that the company needs a new widget for the executive dashboard application.

<span style="font-weight: bold;">&#8230;or we could fake it with a custom loader</span>  
Instead, let&#8217;s create a Hibernate mapping that generates the same results as the view. First, let&#8217;s create a simple POJO to contain the results:

``` java
package us.mikedesjardins;
public class SalesByProduct {
  private Integer id;
  private Integer productId;
  private BigDecimal sales;
// accessors omitted
}
```

The corresponding mapping file would look like this. I re-used the product_id for the ID in this example. Note the loader element:

``` xml
<hibernate-mapping package="us.mikedesjardins">
  <class name="SalesByProduct"
          dynamic-insert="false"
          dynamic-update="false"
          mutable="false">
    <id name="id" type="int" unsaved-value="null">
      <column name="__id" sql-type="int identity" not-null="true" unique="true" />
    </id>
    <property name="productId" type="int">
      <column name="__product_id" not-null="false" />
    </property>
    <property name="sales" type="java.math.BigDecimal">
      <column name="__total_sales" not-null="false" />
    </property>
    <loader query-ref="salesByProductQuery" />
  </class>
  <sql-query name="salesByProductQuery">
    <return class="SalesByProduct" />
    <![CDATA[
    select product_id as __id
         , product_id as __product_id
         , sum(total_sales) as __total_sales
      from YESTERDAY_SALES
     where product_id = :product_id
    group by product_id
   ]]>
  </sql-query>
</hibernate-mapping>
```

Note that you need to put a positional argument in the query, or Hibernate will get nasty about parsing it. Hope that helps!

 [1]: http://mikedesjardins.net/blog/2008/03/pizza-shop-2-totaling-jpa-order-use.html