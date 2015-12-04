---
title: 'Pizza Shop III : JPA Event Listeners'
author: mdesjardins
layout: post
permalink: /2008/03/31/pizza-shop-iii-jpa-event-listeners/
blogger_blog:
  - mikedesjardins.blogspot.com
blogger_author:
  - Mike Desjardinshttp://www.blogger.com/profile/07892630576680251412noreply@blogger.com
blogger_permalink:
  - /2008/03/pizza-shop-iii-jpa-event-listeners.html
categories:
  - blog
tags:
  - hibernate
  - java
  - jpa
  - pizzashop
---
This comment was recently posted to one of my blog entries a while back:

> <span class="PostFooter">Let us consider I am adding &#8216;CreatedBy&#8217; and &#8216;ModifiedBy&#8217; columns in all the tables to be used for audit purposes. Ideally these columns would be populated by the user id who has logged into the application. &#8211; @Madhan</span>

So, let&#8217;s say your data model has some tables with a columns that indicate <img src="/assets/images/1764153258_11bbbcb337-715955.jpg" alt="" border="0" /> the last person who touched a record, like Madhan&#8217;s example above. In most applications, the end-users of the database client share a common login to the database, and have individual logins which are specific to the application&#8217;s domain (i.e., you don&#8217;t have a database login mapped to each end user; maintaining this scheme would be a nightmare). For that reason, triggers can&#8217;t be used as a solution, because database triggers don&#8217;t know anything about which application end-user is responsible for making a data change.

The last thing you want to do is litter your codebase with snippets of code that set the username on your persisted objects manually; not only is it unnecessary duplication, but you&#8217;ll probably also end up missing cases where you should be setting the username.

<span style="font-weight: bold;">EventListeners to the Rescue</span>  
Fortunately, JPA supports the notion of &#8220;EventListeners.&#8221; An event listener intercepts many of the API calls that modify a persisted object&#8217;s lifecycle, and thus may be used to inject business logic that needs to be duplicated over many different objects. AOP aficionados might refer to this as a cross-cutting &#8220;aspect&#8221; of the domain layer.

<span style="font-weight: bold;">Return to the Pizza Shop</span>  
Readers of my blog (all three of them, including me) might recall my venerable Pizza Shop example. Here are the earlier Pizza Shop posts if you&#8217;d like to catch up: [Part 1][1] and [Part 2][2]. I&#8217;m going to drag the Pizza Shop out again to demonstrate how to create an &#8220;audited&#8221; table, which shows the last user who modified a record. As always, the source is available [here][3], and it&#8217;s been tested against MySQL, Postgres, and MS SQL Server, with Hibernate, OpenJPA, and TopLink.

This takes just five easy steps&#8230; four, really, &#8216;cuz the fourth step creates a mock object for testing, so it doesn&#8217;t really count!

<span style="font-weight: bold;">Step One &#8211; New schema</span>  
Add a nullable column named username to the Order table&#8230; something like this should work if you have existing pizzashop schema for some reason:

``` sql
ALTER TABLE ORDER ADD username VARCHAR(10)
```

<span style="font-weight: bold;">Step Two &#8211; Create an Interface and Implement It</span>  
This step isn&#8217;t strictly necessary, but it&#8217;s probably safe to assume that we&#8217;ll want to add this functionality to other tables someday. The interface just adds accessor methods for the username:

``` java
public interface AuditedObject {
  public String getUsername();
  public void setUsername(String username);
}
```

Then, make the Order table implement AuditedObject. Add a member variable with a mapping to the username column to the Order table, and corresponding accessor methods:

``` java
public class Order implements IdObject, AuditedObject {
.
.
  @Basic @Column(name="username")
  private String username;
.
.
  public String getUsername() {
      return username;
  }
  public void setUsername(String username) {
      this.username = username;
  }
.
.
}
```

<span style="font-weight: bold;">Step Three &#8211; Add an EntityListener Annotation</span>  
This is the secret sauce. In the JPA Framework, EventListeners allow you to fire some trigger code when a lifecycle event occurs on a persisted object. You do this by associating your persisted class with an EventListener class. We&#8217;ll implement our EventListener class after we&#8217;ve added the following annotations:

``` java
@Entity @Table(name="PIZZA_ORDER")
@EntityListeners(AuditedEntityListener.class)
public class Order implements IdObject, AuditedObject {
.
.
```

<span style="font-weight: bold;">Step Four &#8211; Write a Mock Object for the Username for Testing</span>  
In the real world, you&#8217;d probably stuff the username into the servlet context object. For our simple tests, we&#8217;ll need to mock up an object to maintain a username for the duration of our tests. It can be something stupidly simple, like this:

``` java
public class MockContext {
  private static String username;
 
  public static String getUsername() {
      return username;
  }
  public static void setUsername(String username) {
      MockContext.username = username;
  }
}
```

We&#8217;ll set the username at the start of our tests, and read the username from our MockContext from the EventListener.

<span style="font-weight: bold;">Step Five &#8211; Write the EntityListener class</span>  
JPA allows you to inercept method calls using seven annotations: 
*   @PrePersist and @PostPersist are called before and after an object is persisted. 
*   @PreUpdate and @PostUpdate are called before and after synchronization with the database. @PreRemove and @PostRemove are called before and after an object is removed from the persistent state.
*   @PostLoad is invoked immediately after an object is loaded from the database.

In our case, we&#8217;re going to populate the username instance variable when the object is persisted, so we will want to create an EntityListener class with the @PrePersist annotation. The method&#8217;s signature takes an Object parameter which is the object getting updated, and returns void. The class will look something like this:

``` java
public class AuditedEntityListener {
  @PrePersist
  public void updateUser(Object o) {
      if (o instanceof AuditedObject) {
        String username = MockContext.getUsername();
        ((AuditedObject)o).setUsername(username);
      }
  }
}
```

<span style="font-style: italic;">Voila</span>! When you run the tests, the username fields are populated and persisted!

Audit M&M&#8217;s Photo Courtesy [Joe Hall][4].

 [1]: http://mikedesjardins.us/blog/2008/01/new-jpa-tutorial-pizza-shop.html
 [2]: http://mikedesjardins.us/blog/2008/03/pizza-shop-2-totaling-jpa-order-use.html
 [3]: http://mikedesjardins.us/pizzashop-3.1.tar.gz
 [4]: http://josephhall.org