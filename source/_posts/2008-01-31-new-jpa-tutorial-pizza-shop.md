---
title: 'A Delicious and Simple JPA Mapping tutorial: The Pizza Shop'
author: mdesjardins
layout: post
permalink: /2008/01/31/new-jpa-tutorial-pizza-shop/
blogger_blog:
  - mikedesjardins.blogspot.com
blogger_author:
  - Mike Desjardinshttp://www.blogger.com/profile/07892630576680251412noreply@blogger.com
blogger_permalink:
  - /2008/01/new-jpa-tutorial-pizza-shop.html
categories:
  - blog
tags:
  - hibernate
  - java
  - jpa
  - pizzashop
  - programming
---
<center>
<img src="/assets/images/pizzasign-715207.jpg" alt="" border="0" />
</center>
So, another JPA tutorial. What makes this one different? Well, for one thing, this one comes with a working, downloadable project that works with Eclipse, NetBeans, and IntelliJ IDEA 7. It&#8217;s packaged with Hibernate, Toplink, and OpenJPA. And it&#8217;s been tested with MySQL, PostgreSQL, MS SQL Server, and Sybase. In other words, it works with 36 different IDE/JPA Provider/Database combinations!

Another thing that makes this tutorial different is the subject matter: Pizza! Who doesn&#8217;t love pizza? Except lactose intolerant people. And people who can&#8217;t eat gluten. But <span style="font-style: italic;">other than them</span>, who doesn&#8217;t love pizza? So we&#8217;re going to create a simple database model for a pizza shop&#8217;s point-of-sale system.

<span style="font-weight: bold;font-size:130%;" >The Schema</span>  
Unsurprisingly, the starting point for any ORM task is usually the database schema (there are people who start with the Objects and work &#8220;backward&#8221; to the schema, but I haven&#8217;t worked with any of them yet). In our example, we have a pristine, consistent, completely normalized schema. In other words, it&#8217;s probably nothing like you&#8217;ll ever be lucky enough to see in the real world! Here&#8217;s our simple little ERD: <img src="/assets/images/pizza-erd-737223.jpg" alt="" border="0" />From this ERD, we can infer the following: 1.) An order is comprised of zero or more pizzas. 2.) A pizza is associated with one size. 3.) A pizza may be associated with a string of text containing &#8220;special instructions.&#8221; 4.) A pizza may have zero or more toppings. You probably also notice that each table has a column called version. This will be used for an [optimistic locking strategy][1].

The first question is, &#8220;Where should we start?&#8221; There&#8217;s no right answer for this, but I find that it&#8217;s easiest to start working with the entities with the fewest dependencies. For example, you can&#8217;t have an order without a pizza, and you can&#8217;t have a pizza without a size, so maybe it makes sense to start with the size. But before that, we&#8217;ll want an ID interface.

<span style="font-size:130%;"><span style="font-weight: bold;">An ID Interface</span></span>  
First things first. You&#8217;ll notice that, in our schema, every table has an integer ID. It&#8217;s often a good idea to have all of your objects implement the same interface for accessing the ID, because it makes it easier to create Generic DAOs (more about that in a future post). For now, let&#8217;s make a really simple interface like this:

``` java
public interface IdObject {
  public void setId(Integer id);
  public Integer getId();
}
```

<span style="font-weight: bold;font-size:130%;" >Many-to-One Unidirectional Relationships</span>  
Now that we&#8217;ve gotten that out of the way, let&#8217;s create the Size class. It&#8217;s a simple POJO littered with annotations, like this:

``` java
@Entity
@Table(name="PIZZA_SIZE")
public class Size implements IdObject {
  @Id
  @Column(name="pizza_size_id")
  private Integer id;
 
  @Column(name="pizza_size_description")
  private String description;
 
  @Column(name="pizza_size_base_price")
  private BigDecimal basePrice;
 
  @Version @Column(name="version")
  private Integer version;
 
  public Integer getId() { return id; }
  public void setId(Integer id) { this.id = id; }
 
  public String getDescription() { return description; }
  public void setDescription(String description) { this.description = description; }
 
  public BigDecimal getBasePrice() { return basePrice; }
  public void setBasePrice(BigDecimal basePrice) { this.basePrice = basePrice; }
 
  public Integer getVersion() { return version; }
  public void setVersion(Integer version) { this.version = version; }
}
```

Here are some notes on the annotations that were used in the above class: <span style="font-weight: bold;"><br />@Entity</span> tells the JPA provider that this is a managed object.  
<span style="font-weight: bold;">@Table</span> specifies the table name. The JPA provider will attempt to default the table name to a sane value based on the class name, but I like to be explicit. I&#8217;m funny that way. Perhaps it&#8217;s [OCD][2]. Or a [power-trip][3].  
<span style="font-weight: bold;">@Column</span> indicates the name of the column, and can include other attributes about the column (you&#8217;ll see a few additional attributes later on). Again, JPA can try to default this to sane values for you, but I like to be explicit.  
<span style="font-weight: bold;">@Version</span> indicates that a particular column is used for indicating when a row is updated. This column can then be used in an [optimistic locking][1] scheme.

Next, let&#8217;s do the Pizza object to show how we map a Pizza to a Size.  
<center>
<img src="/assets/images/iStock_000004523455XSmall-738216.jpg" alt="" border="0" />
</center>

One thing I that always tripped me up when I started out with ORM  
tools was the difference between &#8220;One-to-Many&#8221; and &#8220;Many-to-One.&#8221; I never knew, if I call a relationship many-to-one in my metadata, is <span style="font-style: italic;">this</span> object the one that there are many of, or is it the other way around? The answer is that &#8220;this&#8221; object always comes first. <span>Many</span>ToOne means that &#8220;there are many of <span style="font-style: italic;">this</span> object to one of <span style="font-style: italic;">those</span> objects.&#8221; The &#8220;Many&#8221; side is often the side that has the foreign key.

In our case, there will be many Pizzas that are the same size. So when we make our Pizza object, we will want to use the <span style="font-weight: bold;">@ManyToOne</span> annotation. Here&#8217;s what the Pizza object looks like so far. I&#8217;ve omitted the getters and setters to save space:

``` java
@Entity
@Table(name="PIZZA")
public class Pizza implements IdObject {
  @Id
  @Column(name="pizza_id")
  private Integer id;
 
  @ManyToOne(cascade={CascadeType.ALL})
  @JoinColumn(name="pizza_size_id",nullable=false)
  private Size size;
 
  @Version @Column(name="version")
  private Integer version;
 
  // (Accessor methods omitted)
}
```

Note that, when we made our Size object, we did not include a reference to the Pizza. That was an intentional design decision. In this application, it&#8217;s unlikely that we will want to instantiate a Size object, and get a collection containing all of the Pizzas of that size, so we don&#8217;t bother with mapping it. This is called <span style="font-style: italic;">unidirectional association</span>.

The <span style="font-weight: bold;">@ManyToOne</span> annotation specifies a cascade attribute. There are several different settings for this attribute, which you can read more about [here][4]. I tend to cascade the persistent state to all related objects because it reduces the amount of redundant API calls. By default, JPA does <span style="font-style: italic;">not</span> cascade pers istence to related objects. I&#8217;ll cover the cascade attribute in future posts, but for now, we&#8217;ll go with my personal preference, because I&#8217;m writing the article!

The <span style="font-weight: bold;">@JoinColumn</span> annotation indicates the column name that defines the linkage between the Pizza and the Size. You&#8217;ll also note that we&#8217;ve included some additional attributes on our <span style="font-weight: bold;">@Column</span> and <span style="font-weight: bold;">@JoinColumn</span> annotations. The unique and nullable attributes are particularly useful if you use tools to generate schema DDL from your mappings.

<span style="font-size:130%;"><span style="font-weight: bold;">One-to-Many bidirectional relationships</span></span>  
<center>
<img src="/assets/images/iStock_000004945600XSmall-743564.jpg" alt="" border="0" />
</center>
Both the SpecialInstruction and the Order objects are examples of One-to-Many bidirectional relationships. In the case of SpecialInstruction, it is likely that we will care about which Pizza an instruction is associated with, and likewise for the Order. A bidirectional one-to-many relationship implies that one object has a <span style="font-style: italic;">collection</span> of other o bjects. For example, an Order has a collection of Pizzas. First, lets add an order attribute to our Pizza object:

``` java
@Entity
@Table(name="PIZZA")
public class Pizza implements IdObject {
.
.
  @ManyToOne(cascade={CascadeType.ALL})
  @JoinColumn(name="pizza_order_id",nullable=false)
  private Order order;
 
  public Order getOrder() { return order; }
  public void setOrder(Order order) { this.order = order; }
.
.
}
```

Next, let&#8217;s create an Order object to contain our collection of Pizzas, like this:

``` java
@Entity
@Table(name="PIZZA_ORDER")
public class Order implements IdObject {
  @Id
  @Column(name="pizza_order_id")
  private Integer id;
 
  @OneToMany(cascade={CascadeType.ALL},mappedBy="order")
  private Set pizzas = new HashSet();
 
  @Version @Column(name="version")
  private Integer version;
 
  // (version and id accessors omitted)
 
  public Set getPizzas() {  return pizzas; }
  public void setPizzas(Set pizzas) { this.pizzas = pizzas; }
  public void addPizza(Pizza pizza) {  pizza.setOrder(this);  this.pizzas.add(pizza); }
}
```

There are a couple of things you worth noting about this mapping:  
1.) The <span style="font-weight: bold;">@OneToMany</span> annotation uses the <span style="font-style: italic;">mappedBy</span> attribute to indicate which member of the related object defines the linkage between the two tables. In this case, we are saying that the Pizza object contains a member named order, which defines the linkage between the two objects.  
2.) I&#8217;ve created a utility method called <span style="font-weight: bold;">addPizza</span>. This simplifies setting both sides of the bidirectional relationship by setting the Order object on the Pizza <span style="font-style: italic;">and</span> adding the Pizza to the Order&#8217;s collection. Users of this class will only need to make one method call to do both.

<span style="font-size:130%;"><span style="font-weight: bold;">Many-to-Ma</span></span><span style="font-size:130%;"><span style="font-weight: bold;">ny relationships via a Join Table</span></span>  
<center>
<img src="/assets/images/cohdra100_1404-732719.JPG" alt="" border="0" />
</center>
The last thing we&#8217;ll cover is how to map Join Tables. In our ERD, you can see that Toppings are modeled in the database with a TOPPING table that contains all of the valid toppings, a PIZZA table that contains all of the valid Pizzas, and a PIZZA_TOPPING table in the middle that maps all of the valid Pizzas to all of the valid Toppings. You <span style="font-style: italic;">could</span> create an object called PizzaTopping that corresponds to the PIZZA_TOPPING table. Then you could have a One-to-Many relationship from the Pizza to the PizzaTopping, and a One-to-One from each PizzaTopping to a Topping. That would be very cumbersome to work with! Fortunately, there&#8217;s a better way.

Logically, a Pizza has a collection of Toppings. In our Java code, we really shouldn&#8217;t care about the fact that there is a PIZZA_TOPPING join table in the middle. First, let&#8217;s create a simple Topping class:

``` java
@Entity
@Table(name="TOPPING")
public class Topping implements IdObject {
  @Id
  @Column(name="topping_id")
  private Integer id;
 
  @Column(name="topping_description")
  private String description;
 
  @Column(name="topping_price")
  private BigDecimal price;
 
  @Version @Column(name="version")
  private Integer version;
 
  // (Accessors omitted)
}
```

This is how the association would be mapped in the Pizza class:

``` java
@Entity
@Table(name="PIZZA")
public class Pizza implements IdObject {
.
.
  @ManyToMany(cascade={CascadeType.ALL})
  @JoinTable(name="PIZZA_TOPPING",
             joinColumns=@JoinColumn(name="pizza_id"),
             inverseJoinColumns=@JoinColumn(name="topping_id"))
  private Set toppings = new HashSet();
 
  public Set getToppings() { return toppings; }
  public void setToppings(Set toppings) { this.toppings = toppings; }
  public void addTopping(Topping topping) { this.toppings.add(topping); }
.
.
}
```

The @JoinTable annotation defines three key attributes:  
<span style="font-weight: bold;">name</span>: Identifies the name of the join table.  
<span style="font-weight: bold;">joinColumns</span>: This attribute identifies the column name in the join table that points to <span style="font-style: italic;">this</span> object.  
<span style="font-weight: bold;">inverseJoinColumns</span>: This attribute defines the column name in the join table that points to <span style="font-style: italic;">the other</span> objects.

From your Java code, the semantics for dealing with toppings on a pizza are just like any other set, much like you&#8217;d work with a One-to-many object.

<span style="font-size:130%;"><span style="font-weight: bold;">Conclusion</span></span>  
Since the introduction of annotations, object/relational mapping is one of the easiest aspects of working with JPA over other frameworks. You can see this whole project in action on your IDE of choice by downloading it [here][5] (or from [here][6] if that doesn&#8217;t work for some reason). It should be a pretty reasonable starting point if you want a reference project to start playing with JPA. I hope to revisit the Pizza Shop project to cover other JPA topics in future posts.  

 [1]: http://en.wikipedia.org/wiki/Optimistic_concurrency_control
 [2]: http://en.wikipedia.org/wiki/Obsessive-compulsive_disorder
 [3]: http://en.wikipedia.org/wiki/Dick_cheney
 [4]: http://www.oracle.com/technology/products/ias/toplink/jpa/resources/toplink-jpa-annotations.html#ManyToOne
 [5]: http://www.mediafire.com/?2jgrzkizfy9
 [6]: http://mikedesjardins.us/blog/pizzashop-1.1.tar.gz
