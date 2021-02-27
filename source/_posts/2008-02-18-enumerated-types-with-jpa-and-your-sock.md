---
title: Enumerated Types with JPA, and your Sock Drawer
author: mdesjardins
layout: post
permalink: /2008/02/18/enumerated-types-with-jpa-and-your-sock/
blogger_blog:
  - mikedesjardins.blogspot.com
blogger_permalink:
  - /2008_02_01_archive.html
categories:
  - blog
tags:
  - hibernate
  - java
  - jpa
  - programming
---
<center>
<img src="/assets/images/sockdrawer-773596.jpg" alt="" border="0" />
</center>

One feature of JPA that didn&#8217;t exist in plain-vanilla Hibernate is support for Enumerated types. I haven&#8217;t seen a lot of examples of this in practice or on the internet, so in this post I&#8217;ll show one example of how to use JDK 5 enumerations with JPA.

For our example, we are going to create an inventory system for our sock drawer. It is comprised of only two tables. The first table, called SOCK, contains one row per sock in our drawer. The columns of the table are:

*   sock_id &#8211; an auto-incrementing identity column.
*   sock_description &#8211; a varchar column for a free-form text description of the sock.
*   sock\_pattern\_id &#8211; a reference to a row in the SOCK_PATTERN table.

As you may have guessed, the SOCK_PATTERN table looks like this: 
*   sock\_pattern\_id &#8211; an integer primary key. It&#8217;s not auto-incrementing, because we will want to have control over the contents of the field.
*   sock\_pattern\_description &#8211; a varchar column for a free-form text description of the pattern.

We need to &#8220;prime&#8221; our SOCK_PATTERN table with the valid patterns and create a foreign key relationship between the two tables:

    INSERT INTO SOCK_PATTERN (sock_pattern_id,sock_pattern_description) VALUES (,'SOLID');
    INSERT INTO SOCK_PATTERN (sock_pattern_id,sock_pattern_description) VALUES (1,'STRIPES');
    INSERT INTO SOCK_PATTERN (sock_pattern_id,sock_pattern_description) VALUES (2,'POLKA_DOT');
    INSERT INTO SOCK_PATTERN (sock_pattern_id,sock_pattern_description) VALUES (3,'ARGYLE');

Note that we populated sock\_pattern\_id starting with zero; this is important because the Enumeration below is zero-indexed.

Next, let&#8217;s create the classes:

``` java
public enum SockPattern {
 SOLID, STRIPES, POLKA_DOT, ARGYLE
}
 
@Entity @Table(name="SOCK")
public class Sock {
 @Id @GeneratedValue(strategy=GenerationType.IDENTITY)
 @Column(name="sock_id")
 private Integer id;
 
 @Column(name="sock_description")
 private String description;
 
 @Enumerated @Column(name="sock_pattern_id")
 private SockPattern pattern;
 
 public Integer getId() { return id; }
 public void setId(Integer id) { this.id = id; }
 
 public String getDescription() { return description; }
 public void setDescription(String description) { this.description = description; }
 
 public SockPattern getPattern() { return pattern; }
 public void setPattern(SockPattern pattern) { this.pattern = pattern; }
}
```

Now, to use our wonderful contraption, you&#8217;d do something like the following:

``` java
public class Demo {
 private static EntityManagerFactory emf;
 static {
     Demo.emf = Persistence.createEntityManagerFactory("sockdrawer");
 }
 
 @Test
 public void socksOne() {
     Sock sock = new Sock();
     sock.setDescription("My favorite sock.");
     sock.setPattern(SockPattern.ARGYLE);
     EntityManager em = Demo.emf.createEntityManager();
     em.getTransaction().begin();
     em.persist(sock);
     em.getTransaction().commit();
     em.close();
 }
}
```

There are some obvious pitfalls to this approach. The object/relational mapping is very brittle; for this code to work, the IDs in the database always need to match the values that the ORM tool gets from the enumeration. Changing these values at a later date could cause some surprising results, and you can&#8217;t insert new rows without updating the Enumeration and recompiling. It does obviate the need to map the SOCK_PATTERN table, and you won&#8217;t need to worry about the details of cascading the persistent state of related Sock and SockPattern objects.

It&#8217;s just a new tool in the JPA toolbox.

(The code in this blog post was tested w/ Postgres and Toplink)