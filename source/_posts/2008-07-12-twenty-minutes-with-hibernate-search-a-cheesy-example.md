---
title: Twenty minutes with Hibernate Search; A Cheesy Example
author: mdesjardins
layout: post
permalink: /2008/07/12/twenty-minutes-with-hibernate-search-a-cheesy-example/
blogger_blog:
  - mikedesjardins.blogspot.com
blogger_permalink:
  - /2008_07_01_archive.html
categories:
  - blog
tags:
  - hibernate
  - java
  - jpa
  - programming
---
<center><img src="/assets/images/cheese-blog-719057.jpg" alt="" border="0" /></center>

In a [recent post][1], I showed a trick for determining which users in a system are running a long-running query. One commenter suggested that using a full-text search system made a lot of sense in those situations, and I wholeheartedly agree. So I decided to devote a new post to [Hibernate Search][2]!

In this example, I provide a live, [online interactive search][3] of an online cheese database, and you can download the example (more on that at the end of the post).

<span style="font-weight: bold;">Step One &#8211; The Test Data</span>  
I spent the most time on this project was creating test data for the example program. I headed over to [Freebase][4] to see what was available for data sets. Among lots of other things, they have a free database of [cheese][5]. First, I downloaded the data in TSV format, dumped it into a raw table, and massaged it into a relational, normalized schema. I ended up with this when I was done:
<center>
  <img src="/assets/images/cheese-search-erd-785035.jpg" alt="" border="0" />
</center>
So, a CHEESE can have only one ORIGIN (a country or region where the cheese is made), is made from one-to-many types of MILK, and may have zero-to-many TEXTURES associated with it.

<span style="font-weight: bold;">Step Two &#8211; Build the Domain Model</span>  
The domain model for this system is very simple. Each Cheese class has an Origin, and a set of Milk and Textures:

```java
@Entity @Table(name="MILK")
public class Milk {
  @Id @Column(name="milk_id")
  private Integer id;
 
  @Basic @Column(name="name")
  private String name;
 
  @Version @Column(name="version")
  private Integer version;
 
// Accessors omitted
}
```

```java
@Entity @Table(name="ORIGIN")
public class Origin {
  @Id @Column(name="origin_id")
  private Integer id;
 
  @Basic @Column(name="name")
  private String name;
 
  @Version @Column(name="version")
  private Integer version;
 
// Accessors omitted
}
```

```java
@Entity @Table(name="TEXTURE")
public class Texture {
  @Id @Column(name="texture_id")
  private Integer id;
 
  @Basic @Column(name="description")
  private String description;
 
  @Version @Column(name="version")
  private Integer version;
 
// Accessors omitted
}
```

```java
@Entity @Table(name="CHEESE")
public class Cheese {
  @Id @GeneratedValue(strategy=GenerationType.IDENTITY)
  @Column(name="cheese_id",nullable=false,unique=true)
  private Integer id;
 
  @Basic @Column(name="name")
  private String name;
 
  @ManyToOne(cascade={CascadeType.ALL})
  @JoinColumn(name="origin_id",nullable=false)
  private Origin origin;
 
  @ManyToMany(cascade={CascadeType.PERSIST,CascadeType.MERGE})
  @JoinTable(name="CHEESE_MILK_MAP",
             joinColumns=@JoinColumn(name="cheese_id"),
             inverseJoinColumns=@JoinColumn(name="milk_id"))
  private Set<milk> milks = new HashSet<milk>();
 
  @ManyToMany(cascade={CascadeType.PERSIST,CascadeType.MERGE})
  @JoinTable(name="CHEESE_TEXTURE_MAP",
             joinColumns=@JoinColumn(name="cheese_id"),
             inverseJoinColumns=@JoinColumn(name="texture_id"))
  private Set<texture> textures = new HashSet<texture>();
 
  @Version @Column(name="version")
  private Integer version;
 
// Accessors omitted
}
```

<span style="font-weight: bold;">Step Three &#8211; Add Search Annotations and Configure Lucene</span>  
Behind the scenes, Hibernate Search uses the [Apache Lucene][6] search engine to do its indexing. In short, it maintains a mapping of object IDs to search terms in an external file, and updates the file when objects are added, updated, or deleted. To start using Hibernate Search, you&#8217;ll need to configure the location of these index files, as well as a search directory provider (we&#8217;ll just use the default). This is done in your Hibernate properties file, or (if you use JPA, like me), in persistence.xml:

```xml
<property name="hibernate.search.default.directory_provider" value="org.hibernate.search.store.FSDirectoryProvider" />
<property name="hibernate.search.default.indexBase" value="/var/lucene/cheese-indexes" />
```

Next, you need to indicate to Lucene which classes need to be indexed. You also need to indicate which data fields 1.) contain the document ID, and 2.) contain relevant search text. In our example, we only need to index the Cheese objects. We want to allow users to search on cheese name, milk name, origin, and texture.

First, we indicate that we want to index the Cheese objects by applying the <span style="font-weight: bold;">@Indexed</span> annotation to the class, and we elect to use the Id field to identify the Cheese objects to Lucene by applying a <span style="font-weight: bold;">@DocumentId</span> annotation to it. Next, we indicate that the cheese name contains searchable text by adding the <span style="font-weight: bold;">@Field(index=Index.TOKENIZED, store=Store.NO)</span> annotation to it. The annotation parameters are informing Hibernate Search to use Lucene&#8217;s default tokenizer to summarize the text, and not to store a copy of the document content.

We also want to allow users to search on Origin, Milk Name, and Texture. This text is not contained within the Cheese object, instead they&#8217;re in related object. So we need to add the <span style="font-weight: bold;">@IndexEmbedded</span> annotation to the member variables in the Cheese class that refer to the objects which contain the searchable text, and we also need to add the <span style="font-weight: bold;">@Field(index=Index.TOKENIZED, store=Store.NO)</span> annotation to the Milk, Origin, and Texture classes to indicate which fields are searchable.

When you&#8217;re done, the modified domain classes will look like this:

```java
@Entity @Table(name="MILK")
public class Milk {
  @Id @Column(name="milk_id")
  private Integer id;
 
  @Field(index=Index.TOKENIZED)
  @Basic @Column(name="name")
  private String name;
 
  @Version @Column(name="version")
  private Integer version;
 
// Accessors omitted
}
```

```java
@Entity @Table(name="ORIGIN")
public class Origin {
  @Id @Column(name="origin_id")
  private Integer id;
 
  @Field(index=Index.TOKENIZED)
  @Basic @Column(name="name")
  private String name;
 
  @Version @Column(name="version")
  private Integer version;
 
// Accessors omitted
}
```

```java
@Entity @Table(name="TEXTURE")
public class Texture {
  @Id @Column(name="texture_id")
  private Integer id;
 
  @Field(index=Index.TOKENIZED)
  @Basic @Column(name="description")
  private String description;
 
  @Version @Column(name="version")
  private Integer version;
 
// Accessors omitted
}
```

```java
@Indexed
@Entity @Table(name="CHEESE")
public class Cheese {
  @DocumentId
  @Id @GeneratedValue(strategy=GenerationType.IDENTITY)
  @Column(name="cheese_id",nullable=false,unique=true)
  private Integer id;
 
  @Basic @Column(name="name")
  @Field(index=Index.TOKENIZED, store=Store.NO)
  private String name;
 
  @IndexedEmbedded
  @ManyToOne(cascade={CascadeType.ALL})
  @JoinColumn(name="origin_id",nullable=false)
  private Origin origin;
 
  @IndexedEmbedded
  @ManyToMany(cascade={CascadeType.PERSIST,CascadeType.MERGE})
  @JoinTable(name="CHEESE_MILK_MAP",
             joinColumns=@JoinColumn(name="cheese_id"),
             inverseJoinColumns=@JoinColumn(name="milk_id"))
  private Set<milk> milks = new HashSet<milk>();
 
  @IndexedEmbedded
  @ManyToMany(cascade={CascadeType.PERSIST,CascadeType.MERGE})
  @JoinTable(name="CHEESE_TEXTURE_MAP",
             joinColumns=@JoinColumn(name="cheese_id"),
             inverseJoinColumns=@JoinColumn(name="texture_id"))
  private Set<texture> textures = new HashSet<texture>();
 
  @Version @Column(name="version")
  private Integer version;
 
// Accessors omitted
}
```

<span style="font-weight: bold;">Step Four &#8211; The Servlets</span>  
In this example, I didn&#8217;t want to rely on any web frameworks or even on JSPs, so I wrote a good old-fashioned servlet to exercise the search function. No sane person would ever do it this way. There are actually two servlets in the example &#8211; one for application initialization and one for the page itself.

The initialization servlet does the work of indexing all of the database data the first time through. For our small data set, this takes less than a minute. For larger data sets, it wouldn&#8217;t make sense to re-index everything every time you start the application. The initialization code iterates over all of the Cheese objects ant tells the FullTextEntityManger to index it:

```java
public void init() {
  Dao<cheese> dao = new CheeseDao();
  EntityManager em = dao.getEntityManager();
  FullTextEntityManager fullTextEntityManager = Search.createFullTextEntityManager(em);
 
  List<cheese> cheeses = em.createQuery("select c from Cheese as c").getResultList();
  for (Cheese cheese : cheeses) {
    fullTextEntityManager.index(cheese);
  }
}
```

The main page servlet has some code in the doPost method to perform the search based on the contents of the text form field. That code looks like this (I&#8217;ve shortened it up a bit here by removing some error checking and HTML output):

```java
public void doPost(HttpServletRequest request,
                   HttpServletResponse response) throws ServletException, IOException {
  response.setContentType("text/html");
  PrintWriter out = response.getWriter();
  emitHeader(out);
 
  String searchTerm = request.getParameter("searchterm");
  EntityManager em = dao.getEntityManager();
 
  FullTextEntityManager fullTextEntityManager =
    org.hibernate.search.jpa.Search.createFullTextEntityManager(em);
  MultiFieldQueryParser parser =
    new MultiFieldQueryParser( new String[]{"name",
                                            "origin.name",
                                            "milks.name",
                                            "textures.description"},
                               new StandardAnalyzer());
 
  try {
    org.apache.lucene.search.Query query = parser.parse(searchTerm);
    javax.persistence.Query hibQuery =
      fullTextEntityManager.createFullTextQuery(query,Cheese.class);
    List<cheese> result = hibQuery.getResultList();
    emitTable(out,result);
  } catch (ParseException e) {
    log.error("Got a parse exception", e);
    throw new ServletException(e.getMessage());
  }
  emitFooter(out);
  out.close();
}
```

That&#8217;s all there is to it! As you can see, setting up Hibernate Search is very simple. Most of the effort for this project was spent creating the data and making the servlets work.

<span style="font-weight: bold;">Enjoy the Finished Product</span>  
To see this less-than-world-changing application in action, visit it [here][3]. You can also [download the whole eclipse project][7] and try it out for yourself. It comes with SQL dumps suitable for MySQL and PostgreSQL, and it has been tested with both environments under Tomcat 5.5.

<span style="font-style: italic;">Photo Credit: </span>[<span style="font-style: italic;font-size:100%;" ><span class="RealName"><span class="fn n"><span class="given-name">Chris</span> <span class="family-name">Buecheler</span></span></span></span>][8]

 [1]: http://mikedesjardins.net/blog/2008/06/who-is-running-that-horrible-hibernate.html
 [2]: http://www.hibernate.org/410.html
 [3]: http://mikedesjardins.us/cheese/search
 [4]: http://freebase.com/
 [5]: http://www.freebase.com/view/food/cheese
 [6]: http://lucene.apache.org/
 [7]: http://mikedesjardins.us/cheese-search-1.0.tar.gz
 [8]: http://www.flickr.com/people/cuse/