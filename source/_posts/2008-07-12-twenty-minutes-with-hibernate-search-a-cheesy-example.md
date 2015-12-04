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

<div class="wp_syntax">
  <div class="code">
    <pre class="java" style="font-family:monospace;">@<span style="color: #003399;">Entity</span> @Table<span style="color: #009900;">&#40;</span>name<span style="color: #339933;">=</span><span style="color: #0000ff;">"MILK"</span><span style="color: #009900;">&#41;</span>
<span style="color: #000000; font-weight: bold;">public</span> <span style="color: #000000; font-weight: bold;">class</span> Milk <span style="color: #009900;">&#123;</span>
  @Id @Column<span style="color: #009900;">&#40;</span>name<span style="color: #339933;">=</span><span style="color: #0000ff;">"milk_id"</span><span style="color: #009900;">&#41;</span>
  <span style="color: #000000; font-weight: bold;">private</span> <span style="color: #003399;">Integer</span> id<span style="color: #339933;">;</span>
&nbsp;
  @Basic @Column<span style="color: #009900;">&#40;</span>name<span style="color: #339933;">=</span><span style="color: #0000ff;">"name"</span><span style="color: #009900;">&#41;</span>
  <span style="color: #000000; font-weight: bold;">private</span> <span style="color: #003399;">String</span> name<span style="color: #339933;">;</span>
&nbsp;
  @Version @Column<span style="color: #009900;">&#40;</span>name<span style="color: #339933;">=</span><span style="color: #0000ff;">"version"</span><span style="color: #009900;">&#41;</span>
  <span style="color: #000000; font-weight: bold;">private</span> <span style="color: #003399;">Integer</span> version<span style="color: #339933;">;</span>
&nbsp;
<span style="color: #666666; font-style: italic;">// Accessors omitted</span>
<span style="color: #009900;">&#125;</span></pre>
  </div>
</div>



<div class="wp_syntax">
  <div class="code">
    <pre class="java" style="font-family:monospace;">@<span style="color: #003399;">Entity</span> @Table<span style="color: #009900;">&#40;</span>name<span style="color: #339933;">=</span><span style="color: #0000ff;">"ORIGIN"</span><span style="color: #009900;">&#41;</span>
<span style="color: #000000; font-weight: bold;">public</span> <span style="color: #000000; font-weight: bold;">class</span> Origin <span style="color: #009900;">&#123;</span>
  @Id @Column<span style="color: #009900;">&#40;</span>name<span style="color: #339933;">=</span><span style="color: #0000ff;">"origin_id"</span><span style="color: #009900;">&#41;</span>
  <span style="color: #000000; font-weight: bold;">private</span> <span style="color: #003399;">Integer</span> id<span style="color: #339933;">;</span>
&nbsp;
  @Basic @Column<span style="color: #009900;">&#40;</span>name<span style="color: #339933;">=</span><span style="color: #0000ff;">"name"</span><span style="color: #009900;">&#41;</span>
  <span style="color: #000000; font-weight: bold;">private</span> <span style="color: #003399;">String</span> name<span style="color: #339933;">;</span>
&nbsp;
  @Version @Column<span style="color: #009900;">&#40;</span>name<span style="color: #339933;">=</span><span style="color: #0000ff;">"version"</span><span style="color: #009900;">&#41;</span>
  <span style="color: #000000; font-weight: bold;">private</span> <span style="color: #003399;">Integer</span> version<span style="color: #339933;">;</span>
&nbsp;
<span style="color: #666666; font-style: italic;">// Accessors omitted</span>
<span style="color: #009900;">&#125;</span></pre>
  </div>
</div>



<div class="wp_syntax">
  <div class="code">
    <pre class="java" style="font-family:monospace;">@<span style="color: #003399;">Entity</span> @Table<span style="color: #009900;">&#40;</span>name<span style="color: #339933;">=</span><span style="color: #0000ff;">"TEXTURE"</span><span style="color: #009900;">&#41;</span>
<span style="color: #000000; font-weight: bold;">public</span> <span style="color: #000000; font-weight: bold;">class</span> Texture <span style="color: #009900;">&#123;</span>
  @Id @Column<span style="color: #009900;">&#40;</span>name<span style="color: #339933;">=</span><span style="color: #0000ff;">"texture_id"</span><span style="color: #009900;">&#41;</span>
  <span style="color: #000000; font-weight: bold;">private</span> <span style="color: #003399;">Integer</span> id<span style="color: #339933;">;</span>
&nbsp;
  @Basic @Column<span style="color: #009900;">&#40;</span>name<span style="color: #339933;">=</span><span style="color: #0000ff;">"description"</span><span style="color: #009900;">&#41;</span>
  <span style="color: #000000; font-weight: bold;">private</span> <span style="color: #003399;">String</span> description<span style="color: #339933;">;</span>
&nbsp;
  @Version @Column<span style="color: #009900;">&#40;</span>name<span style="color: #339933;">=</span><span style="color: #0000ff;">"version"</span><span style="color: #009900;">&#41;</span>
  <span style="color: #000000; font-weight: bold;">private</span> <span style="color: #003399;">Integer</span> version<span style="color: #339933;">;</span>
&nbsp;
<span style="color: #666666; font-style: italic;">// Accessors omitted</span>
<span style="color: #009900;">&#125;</span></pre>
  </div>
</div>



<div class="wp_syntax">
  <div class="code">
    <pre class="java" style="font-family:monospace;">@<span style="color: #003399;">Entity</span> @Table<span style="color: #009900;">&#40;</span>name<span style="color: #339933;">=</span><span style="color: #0000ff;">"CHEESE"</span><span style="color: #009900;">&#41;</span>
<span style="color: #000000; font-weight: bold;">public</span> <span style="color: #000000; font-weight: bold;">class</span> Cheese <span style="color: #009900;">&#123;</span>
  @Id @GeneratedValue<span style="color: #009900;">&#40;</span>strategy<span style="color: #339933;">=</span>GenerationType.<span style="color: #006633;">IDENTITY</span><span style="color: #009900;">&#41;</span>
  @Column<span style="color: #009900;">&#40;</span>name<span style="color: #339933;">=</span><span style="color: #0000ff;">"cheese_id"</span>,nullable<span style="color: #339933;">=</span><span style="color: #000066; font-weight: bold;">false</span>,unique<span style="color: #339933;">=</span><span style="color: #000066; font-weight: bold;">true</span><span style="color: #009900;">&#41;</span>
  <span style="color: #000000; font-weight: bold;">private</span> <span style="color: #003399;">Integer</span> id<span style="color: #339933;">;</span>
&nbsp;
  @Basic @Column<span style="color: #009900;">&#40;</span>name<span style="color: #339933;">=</span><span style="color: #0000ff;">"name"</span><span style="color: #009900;">&#41;</span>
  <span style="color: #000000; font-weight: bold;">private</span> <span style="color: #003399;">String</span> name<span style="color: #339933;">;</span>
&nbsp;
  @ManyToOne<span style="color: #009900;">&#40;</span>cascade<span style="color: #339933;">=</span><span style="color: #009900;">&#123;</span>CascadeType.<span style="color: #006633;">ALL</span><span style="color: #009900;">&#125;</span><span style="color: #009900;">&#41;</span>
  @JoinColumn<span style="color: #009900;">&#40;</span>name<span style="color: #339933;">=</span><span style="color: #0000ff;">"origin_id"</span>,nullable<span style="color: #339933;">=</span><span style="color: #000066; font-weight: bold;">false</span><span style="color: #009900;">&#41;</span>
  <span style="color: #000000; font-weight: bold;">private</span> Origin origin<span style="color: #339933;">;</span>
&nbsp;
  @ManyToMany<span style="color: #009900;">&#40;</span>cascade<span style="color: #339933;">=</span><span style="color: #009900;">&#123;</span>CascadeType.<span style="color: #006633;">PERSIST</span>,CascadeType.<span style="color: #006633;">MERGE</span><span style="color: #009900;">&#125;</span><span style="color: #009900;">&#41;</span>
  @JoinTable<span style="color: #009900;">&#40;</span>name<span style="color: #339933;">=</span><span style="color: #0000ff;">"CHEESE_MILK_MAP"</span>,
             joinColumns<span style="color: #339933;">=</span>@JoinColumn<span style="color: #009900;">&#40;</span>name<span style="color: #339933;">=</span><span style="color: #0000ff;">"cheese_id"</span><span style="color: #009900;">&#41;</span>,
             inverseJoinColumns<span style="color: #339933;">=</span>@JoinColumn<span style="color: #009900;">&#40;</span>name<span style="color: #339933;">=</span><span style="color: #0000ff;">"milk_id"</span><span style="color: #009900;">&#41;</span><span style="color: #009900;">&#41;</span>
  <span style="color: #000000; font-weight: bold;">private</span> Set<span style="color: #339933;">&lt;</span>milk<span style="color: #339933;">&gt;</span> milks <span style="color: #339933;">=</span> <span style="color: #000000; font-weight: bold;">new</span> HashSet<span style="color: #339933;">&lt;</span>milk<span style="color: #339933;">&gt;</span><span style="color: #009900;">&#40;</span><span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
&nbsp;
  @ManyToMany<span style="color: #009900;">&#40;</span>cascade<span style="color: #339933;">=</span><span style="color: #009900;">&#123;</span>CascadeType.<span style="color: #006633;">PERSIST</span>,CascadeType.<span style="color: #006633;">MERGE</span><span style="color: #009900;">&#125;</span><span style="color: #009900;">&#41;</span>
  @JoinTable<span style="color: #009900;">&#40;</span>name<span style="color: #339933;">=</span><span style="color: #0000ff;">"CHEESE_TEXTURE_MAP"</span>,
             joinColumns<span style="color: #339933;">=</span>@JoinColumn<span style="color: #009900;">&#40;</span>name<span style="color: #339933;">=</span><span style="color: #0000ff;">"cheese_id"</span><span style="color: #009900;">&#41;</span>,
             inverseJoinColumns<span style="color: #339933;">=</span>@JoinColumn<span style="color: #009900;">&#40;</span>name<span style="color: #339933;">=</span><span style="color: #0000ff;">"texture_id"</span><span style="color: #009900;">&#41;</span><span style="color: #009900;">&#41;</span>
  <span style="color: #000000; font-weight: bold;">private</span> Set<span style="color: #339933;">&lt;</span>texture<span style="color: #339933;">&gt;</span> textures <span style="color: #339933;">=</span> <span style="color: #000000; font-weight: bold;">new</span> HashSet<span style="color: #339933;">&lt;</span>texture<span style="color: #339933;">&gt;</span><span style="color: #009900;">&#40;</span><span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
&nbsp;
  @Version @Column<span style="color: #009900;">&#40;</span>name<span style="color: #339933;">=</span><span style="color: #0000ff;">"version"</span><span style="color: #009900;">&#41;</span>
  <span style="color: #000000; font-weight: bold;">private</span> <span style="color: #003399;">Integer</span> version<span style="color: #339933;">;</span>
&nbsp;
<span style="color: #666666; font-style: italic;">// Accessors omitted</span>
<span style="color: #009900;">&#125;</span></pre>
  </div>
</div>

<span style="font-weight: bold;">Step Three &#8211; Add Search Annotations and Configure Lucene</span>  
Behind the scenes, Hibernate Search uses the [Apache Lucene][6] search engine to do its indexing. In short, it maintains a mapping of object IDs to search terms in an external file, and updates the file when objects are added, updated, or deleted. To start using Hibernate Search, you&#8217;ll need to configure the location of these index files, as well as a search directory provider (we&#8217;ll just use the default). This is done in your Hibernate properties file, or (if you use JPA, like me), in persistence.xml:

<div class="wp_syntax">
  <div class="code">
    <pre class="xml" style="font-family:monospace;"><span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;property</span> <span style="color: #000066;">name</span>=<span style="color: #ff0000;">"hibernate.search.default.directory_provider"</span> <span style="color: #000066;">value</span>=<span style="color: #ff0000;">"org.hibernate.search.store.FSDirectoryProvider"</span> <span style="color: #000000; font-weight: bold;">/&gt;</span></span>
<span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;property</span> <span style="color: #000066;">name</span>=<span style="color: #ff0000;">"hibernate.search.default.indexBase"</span> <span style="color: #000066;">value</span>=<span style="color: #ff0000;">"/var/lucene/cheese-indexes"</span> <span style="color: #000000; font-weight: bold;">/&gt;</span></span></pre>
  </div>
</div>

Next, you need to indicate to Lucene which classes need to be indexed. You also need to indicate which data fields 1.) contain the document ID, and 2.) contain relevant search text. In our example, we only need to index the Cheese objects. We want to allow users to search on cheese name, milk name, origin, and texture.

First, we indicate that we want to index the Cheese objects by applying the <span style="font-weight: bold;">@Indexed</span> annotation to the class, and we elect to use the Id field to identify the Cheese objects to Lucene by applying a <span style="font-weight: bold;">@DocumentId</span> annotation to it. Next, we indicate that the cheese name contains searchable text by adding the <span style="font-weight: bold;">@Field(index=Index.TOKENIZED, store=Store.NO)</span> annotation to it. The annotation parameters are informing Hibernate Search to use Lucene&#8217;s default tokenizer to summarize the text, and not to store a copy of the document content.

We also want to allow users to search on Origin, Milk Name, and Texture. This text is not contained within the Cheese object, instead they&#8217;re in related object. So we need to add the <span style="font-weight: bold;">@IndexEmbedded</span> annotation to the member variables in the Cheese class that refer to the objects which contain the searchable text, and we also need to add the <span style="font-weight: bold;">@Field(index=Index.TOKENIZED, store=Store.NO)</span> annotation to the Milk, Origin, and Texture classes to indicate which fields are searchable.

When you&#8217;re done, the modified domain classes will look like this:

<div class="wp_syntax">
  <div class="code">
    <pre class="java" style="font-family:monospace;">@<span style="color: #003399;">Entity</span> @Table<span style="color: #009900;">&#40;</span>name<span style="color: #339933;">=</span><span style="color: #0000ff;">"MILK"</span><span style="color: #009900;">&#41;</span>
<span style="color: #000000; font-weight: bold;">public</span> <span style="color: #000000; font-weight: bold;">class</span> Milk <span style="color: #009900;">&#123;</span>
  @Id @Column<span style="color: #009900;">&#40;</span>name<span style="color: #339933;">=</span><span style="color: #0000ff;">"milk_id"</span><span style="color: #009900;">&#41;</span>
  <span style="color: #000000; font-weight: bold;">private</span> <span style="color: #003399;">Integer</span> id<span style="color: #339933;">;</span>
&nbsp;
  @<span style="color: #003399;">Field</span><span style="color: #009900;">&#40;</span>index<span style="color: #339933;">=</span>Index.<span style="color: #006633;">TOKENIZED</span><span style="color: #009900;">&#41;</span>
  @Basic @Column<span style="color: #009900;">&#40;</span>name<span style="color: #339933;">=</span><span style="color: #0000ff;">"name"</span><span style="color: #009900;">&#41;</span>
  <span style="color: #000000; font-weight: bold;">private</span> <span style="color: #003399;">String</span> name<span style="color: #339933;">;</span>
&nbsp;
  @Version @Column<span style="color: #009900;">&#40;</span>name<span style="color: #339933;">=</span><span style="color: #0000ff;">"version"</span><span style="color: #009900;">&#41;</span>
  <span style="color: #000000; font-weight: bold;">private</span> <span style="color: #003399;">Integer</span> version<span style="color: #339933;">;</span>
&nbsp;
<span style="color: #666666; font-style: italic;">// Accessors omitted</span>
<span style="color: #009900;">&#125;</span></pre>
  </div>
</div>



<div class="wp_syntax">
  <div class="code">
    <pre class="java" style="font-family:monospace;">@<span style="color: #003399;">Entity</span> @Table<span style="color: #009900;">&#40;</span>name<span style="color: #339933;">=</span><span style="color: #0000ff;">"ORIGIN"</span><span style="color: #009900;">&#41;</span>
<span style="color: #000000; font-weight: bold;">public</span> <span style="color: #000000; font-weight: bold;">class</span> Origin <span style="color: #009900;">&#123;</span>
  @Id @Column<span style="color: #009900;">&#40;</span>name<span style="color: #339933;">=</span><span style="color: #0000ff;">"origin_id"</span><span style="color: #009900;">&#41;</span>
  <span style="color: #000000; font-weight: bold;">private</span> <span style="color: #003399;">Integer</span> id<span style="color: #339933;">;</span>
&nbsp;
  @<span style="color: #003399;">Field</span><span style="color: #009900;">&#40;</span>index<span style="color: #339933;">=</span>Index.<span style="color: #006633;">TOKENIZED</span><span style="color: #009900;">&#41;</span>
  @Basic @Column<span style="color: #009900;">&#40;</span>name<span style="color: #339933;">=</span><span style="color: #0000ff;">"name"</span><span style="color: #009900;">&#41;</span>
  <span style="color: #000000; font-weight: bold;">private</span> <span style="color: #003399;">String</span> name<span style="color: #339933;">;</span>
&nbsp;
  @Version @Column<span style="color: #009900;">&#40;</span>name<span style="color: #339933;">=</span><span style="color: #0000ff;">"version"</span><span style="color: #009900;">&#41;</span>
  <span style="color: #000000; font-weight: bold;">private</span> <span style="color: #003399;">Integer</span> version<span style="color: #339933;">;</span>
&nbsp;
<span style="color: #666666; font-style: italic;">// Accessors omitted</span>
<span style="color: #009900;">&#125;</span></pre>
  </div>
</div>



<div class="wp_syntax">
  <div class="code">
    <pre class="java" style="font-family:monospace;">@<span style="color: #003399;">Entity</span> @Table<span style="color: #009900;">&#40;</span>name<span style="color: #339933;">=</span><span style="color: #0000ff;">"TEXTURE"</span><span style="color: #009900;">&#41;</span>
<span style="color: #000000; font-weight: bold;">public</span> <span style="color: #000000; font-weight: bold;">class</span> Texture <span style="color: #009900;">&#123;</span>
  @Id @Column<span style="color: #009900;">&#40;</span>name<span style="color: #339933;">=</span><span style="color: #0000ff;">"texture_id"</span><span style="color: #009900;">&#41;</span>
  <span style="color: #000000; font-weight: bold;">private</span> <span style="color: #003399;">Integer</span> id<span style="color: #339933;">;</span>
&nbsp;
  @<span style="color: #003399;">Field</span><span style="color: #009900;">&#40;</span>index<span style="color: #339933;">=</span>Index.<span style="color: #006633;">TOKENIZED</span><span style="color: #009900;">&#41;</span>
  @Basic @Column<span style="color: #009900;">&#40;</span>name<span style="color: #339933;">=</span><span style="color: #0000ff;">"description"</span><span style="color: #009900;">&#41;</span>
  <span style="color: #000000; font-weight: bold;">private</span> <span style="color: #003399;">String</span> description<span style="color: #339933;">;</span>
&nbsp;
  @Version @Column<span style="color: #009900;">&#40;</span>name<span style="color: #339933;">=</span><span style="color: #0000ff;">"version"</span><span style="color: #009900;">&#41;</span>
  <span style="color: #000000; font-weight: bold;">private</span> <span style="color: #003399;">Integer</span> version<span style="color: #339933;">;</span>
&nbsp;
<span style="color: #666666; font-style: italic;">// Accessors omitted</span>
<span style="color: #009900;">&#125;</span></pre>
  </div>
</div>



<div class="wp_syntax">
  <div class="code">
    <pre class="java" style="font-family:monospace;">@Indexed
@<span style="color: #003399;">Entity</span> @Table<span style="color: #009900;">&#40;</span>name<span style="color: #339933;">=</span><span style="color: #0000ff;">"CHEESE"</span><span style="color: #009900;">&#41;</span>
<span style="color: #000000; font-weight: bold;">public</span> <span style="color: #000000; font-weight: bold;">class</span> Cheese <span style="color: #009900;">&#123;</span>
  @DocumentId
  @Id @GeneratedValue<span style="color: #009900;">&#40;</span>strategy<span style="color: #339933;">=</span>GenerationType.<span style="color: #006633;">IDENTITY</span><span style="color: #009900;">&#41;</span>
  @Column<span style="color: #009900;">&#40;</span>name<span style="color: #339933;">=</span><span style="color: #0000ff;">"cheese_id"</span>,nullable<span style="color: #339933;">=</span><span style="color: #000066; font-weight: bold;">false</span>,unique<span style="color: #339933;">=</span><span style="color: #000066; font-weight: bold;">true</span><span style="color: #009900;">&#41;</span>
  <span style="color: #000000; font-weight: bold;">private</span> <span style="color: #003399;">Integer</span> id<span style="color: #339933;">;</span>
&nbsp;
  @Basic @Column<span style="color: #009900;">&#40;</span>name<span style="color: #339933;">=</span><span style="color: #0000ff;">"name"</span><span style="color: #009900;">&#41;</span>
  @<span style="color: #003399;">Field</span><span style="color: #009900;">&#40;</span>index<span style="color: #339933;">=</span>Index.<span style="color: #006633;">TOKENIZED</span>, store<span style="color: #339933;">=</span>Store.<span style="color: #006633;">NO</span><span style="color: #009900;">&#41;</span>
  <span style="color: #000000; font-weight: bold;">private</span> <span style="color: #003399;">String</span> name<span style="color: #339933;">;</span>
&nbsp;
  @IndexedEmbedded
  @ManyToOne<span style="color: #009900;">&#40;</span>cascade<span style="color: #339933;">=</span><span style="color: #009900;">&#123;</span>CascadeType.<span style="color: #006633;">ALL</span><span style="color: #009900;">&#125;</span><span style="color: #009900;">&#41;</span>
  @JoinColumn<span style="color: #009900;">&#40;</span>name<span style="color: #339933;">=</span><span style="color: #0000ff;">"origin_id"</span>,nullable<span style="color: #339933;">=</span><span style="color: #000066; font-weight: bold;">false</span><span style="color: #009900;">&#41;</span>
  <span style="color: #000000; font-weight: bold;">private</span> Origin origin<span style="color: #339933;">;</span>
&nbsp;
  @IndexedEmbedded
  @ManyToMany<span style="color: #009900;">&#40;</span>cascade<span style="color: #339933;">=</span><span style="color: #009900;">&#123;</span>CascadeType.<span style="color: #006633;">PERSIST</span>,CascadeType.<span style="color: #006633;">MERGE</span><span style="color: #009900;">&#125;</span><span style="color: #009900;">&#41;</span>
  @JoinTable<span style="color: #009900;">&#40;</span>name<span style="color: #339933;">=</span><span style="color: #0000ff;">"CHEESE_MILK_MAP"</span>,
             joinColumns<span style="color: #339933;">=</span>@JoinColumn<span style="color: #009900;">&#40;</span>name<span style="color: #339933;">=</span><span style="color: #0000ff;">"cheese_id"</span><span style="color: #009900;">&#41;</span>,
             inverseJoinColumns<span style="color: #339933;">=</span>@JoinColumn<span style="color: #009900;">&#40;</span>name<span style="color: #339933;">=</span><span style="color: #0000ff;">"milk_id"</span><span style="color: #009900;">&#41;</span><span style="color: #009900;">&#41;</span>
  <span style="color: #000000; font-weight: bold;">private</span> Set<span style="color: #339933;">&lt;</span>milk<span style="color: #339933;">&gt;</span> milks <span style="color: #339933;">=</span> <span style="color: #000000; font-weight: bold;">new</span> HashSet<span style="color: #339933;">&lt;</span>milk<span style="color: #339933;">&gt;</span><span style="color: #009900;">&#40;</span><span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
&nbsp;
  @IndexedEmbedded
  @ManyToMany<span style="color: #009900;">&#40;</span>cascade<span style="color: #339933;">=</span><span style="color: #009900;">&#123;</span>CascadeType.<span style="color: #006633;">PERSIST</span>,CascadeType.<span style="color: #006633;">MERGE</span><span style="color: #009900;">&#125;</span><span style="color: #009900;">&#41;</span>
  @JoinTable<span style="color: #009900;">&#40;</span>name<span style="color: #339933;">=</span><span style="color: #0000ff;">"CHEESE_TEXTURE_MAP"</span>,
             joinColumns<span style="color: #339933;">=</span>@JoinColumn<span style="color: #009900;">&#40;</span>name<span style="color: #339933;">=</span><span style="color: #0000ff;">"cheese_id"</span><span style="color: #009900;">&#41;</span>,
             inverseJoinColumns<span style="color: #339933;">=</span>@JoinColumn<span style="color: #009900;">&#40;</span>name<span style="color: #339933;">=</span><span style="color: #0000ff;">"texture_id"</span><span style="color: #009900;">&#41;</span><span style="color: #009900;">&#41;</span>
  <span style="color: #000000; font-weight: bold;">private</span> Set<span style="color: #339933;">&lt;</span>texture<span style="color: #339933;">&gt;</span> textures <span style="color: #339933;">=</span> <span style="color: #000000; font-weight: bold;">new</span> HashSet<span style="color: #339933;">&lt;</span>texture<span style="color: #339933;">&gt;</span><span style="color: #009900;">&#40;</span><span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
&nbsp;
  @Version @Column<span style="color: #009900;">&#40;</span>name<span style="color: #339933;">=</span><span style="color: #0000ff;">"version"</span><span style="color: #009900;">&#41;</span>
  <span style="color: #000000; font-weight: bold;">private</span> <span style="color: #003399;">Integer</span> version<span style="color: #339933;">;</span>
&nbsp;
<span style="color: #666666; font-style: italic;">// Accessors omitted</span>
<span style="color: #009900;">&#125;</span></pre>
  </div>
</div>

<span style="font-weight: bold;">Step Four &#8211; The Servlets</span>  
In this example, I didn&#8217;t want to rely on any web frameworks or even on JSPs, so I wrote a good old-fashioned servlet to exercise the search function. No sane person would ever do it this way. There are actually two servlets in the example &#8211; one for application initialization and one for the page itself.

The initialization servlet does the work of indexing all of the database data the first time through. For our small data set, this takes less than a minute. For larger data sets, it wouldn&#8217;t make sense to re-index everything every time you start the application. The initialization code iterates over all of the Cheese objects ant tells the FullTextEntityManger to index it:

<div class="wp_syntax">
  <div class="code">
    <pre class="java" style="font-family:monospace;"><span style="color: #000000; font-weight: bold;">public</span> <span style="color: #000066; font-weight: bold;">void</span> init<span style="color: #009900;">&#40;</span><span style="color: #009900;">&#41;</span> <span style="color: #009900;">&#123;</span>
  Dao<span style="color: #339933;">&lt;</span>cheese<span style="color: #339933;">&gt;</span> dao <span style="color: #339933;">=</span> <span style="color: #000000; font-weight: bold;">new</span> CheeseDao<span style="color: #009900;">&#40;</span><span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
  EntityManager em <span style="color: #339933;">=</span> dao.<span style="color: #006633;">getEntityManager</span><span style="color: #009900;">&#40;</span><span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
  FullTextEntityManager fullTextEntityManager <span style="color: #339933;">=</span> Search.<span style="color: #006633;">createFullTextEntityManager</span><span style="color: #009900;">&#40;</span>em<span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
&nbsp;
  List<span style="color: #339933;">&lt;</span>cheese<span style="color: #339933;">&gt;</span> cheeses <span style="color: #339933;">=</span> em.<span style="color: #006633;">createQuery</span><span style="color: #009900;">&#40;</span><span style="color: #0000ff;">"select c from Cheese as c"</span><span style="color: #009900;">&#41;</span>.<span style="color: #006633;">getResultList</span><span style="color: #009900;">&#40;</span><span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
  <span style="color: #000000; font-weight: bold;">for</span> <span style="color: #009900;">&#40;</span>Cheese cheese <span style="color: #339933;">:</span> cheeses<span style="color: #009900;">&#41;</span> <span style="color: #009900;">&#123;</span>
    fullTextEntityManager.<span style="color: #006633;">index</span><span style="color: #009900;">&#40;</span>cheese<span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
  <span style="color: #009900;">&#125;</span>
<span style="color: #009900;">&#125;</span></pre>
  </div>
</div>

The main page servlet has some code in the doPost method to perform the search based on the contents of the text form field. That code looks like this (I&#8217;ve shortened it up a bit here by removing some error checking and HTML output):

<div class="wp_syntax">
  <div class="code">
    <pre class="java" style="font-family:monospace;"><span style="color: #000000; font-weight: bold;">public</span> <span style="color: #000066; font-weight: bold;">void</span> doPost<span style="color: #009900;">&#40;</span>HttpServletRequest request,
                   HttpServletResponse response<span style="color: #009900;">&#41;</span> <span style="color: #000000; font-weight: bold;">throws</span> ServletException, <span style="color: #003399;">IOException</span> <span style="color: #009900;">&#123;</span>
  response.<span style="color: #006633;">setContentType</span><span style="color: #009900;">&#40;</span><span style="color: #0000ff;">"text/html"</span><span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
  <span style="color: #003399;">PrintWriter</span> out <span style="color: #339933;">=</span> response.<span style="color: #006633;">getWriter</span><span style="color: #009900;">&#40;</span><span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
  emitHeader<span style="color: #009900;">&#40;</span>out<span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
&nbsp;
  <span style="color: #003399;">String</span> searchTerm <span style="color: #339933;">=</span> request.<span style="color: #006633;">getParameter</span><span style="color: #009900;">&#40;</span><span style="color: #0000ff;">"searchterm"</span><span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
  EntityManager em <span style="color: #339933;">=</span> dao.<span style="color: #006633;">getEntityManager</span><span style="color: #009900;">&#40;</span><span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
&nbsp;
  FullTextEntityManager fullTextEntityManager <span style="color: #339933;">=</span>
    org.<span style="color: #006633;">hibernate</span>.<span style="color: #006633;">search</span>.<span style="color: #006633;">jpa</span>.<span style="color: #006633;">Search</span>.<span style="color: #006633;">createFullTextEntityManager</span><span style="color: #009900;">&#40;</span>em<span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
  MultiFieldQueryParser parser <span style="color: #339933;">=</span>
    <span style="color: #000000; font-weight: bold;">new</span> MultiFieldQueryParser<span style="color: #009900;">&#40;</span> <span style="color: #000000; font-weight: bold;">new</span> <span style="color: #003399;">String</span><span style="color: #009900;">&#91;</span><span style="color: #009900;">&#93;</span><span style="color: #009900;">&#123;</span><span style="color: #0000ff;">"name"</span>,
                                            <span style="color: #0000ff;">"origin.name"</span>,
                                            <span style="color: #0000ff;">"milks.name"</span>,
                                            <span style="color: #0000ff;">"textures.description"</span><span style="color: #009900;">&#125;</span>,
                               <span style="color: #000000; font-weight: bold;">new</span> StandardAnalyzer<span style="color: #009900;">&#40;</span><span style="color: #009900;">&#41;</span><span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
&nbsp;
  <span style="color: #000000; font-weight: bold;">try</span> <span style="color: #009900;">&#123;</span>
    org.<span style="color: #006633;">apache</span>.<span style="color: #006633;">lucene</span>.<span style="color: #006633;">search</span>.<span style="color: #006633;">Query</span> query <span style="color: #339933;">=</span> parser.<span style="color: #006633;">parse</span><span style="color: #009900;">&#40;</span>searchTerm<span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
    javax.<span style="color: #006633;">persistence</span>.<span style="color: #006633;">Query</span> hibQuery <span style="color: #339933;">=</span>
      fullTextEntityManager.<span style="color: #006633;">createFullTextQuery</span><span style="color: #009900;">&#40;</span>query,Cheese.<span style="color: #000000; font-weight: bold;">class</span><span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
    List<span style="color: #339933;">&lt;</span>cheese<span style="color: #339933;">&gt;</span> result <span style="color: #339933;">=</span> hibQuery.<span style="color: #006633;">getResultList</span><span style="color: #009900;">&#40;</span><span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
    emitTable<span style="color: #009900;">&#40;</span>out,result<span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
  <span style="color: #009900;">&#125;</span> <span style="color: #000000; font-weight: bold;">catch</span> <span style="color: #009900;">&#40;</span><span style="color: #003399;">ParseException</span> e<span style="color: #009900;">&#41;</span> <span style="color: #009900;">&#123;</span>
    log.<span style="color: #006633;">error</span><span style="color: #009900;">&#40;</span><span style="color: #0000ff;">"Got a parse exception"</span>, e<span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
    <span style="color: #000000; font-weight: bold;">throw</span> <span style="color: #000000; font-weight: bold;">new</span> ServletException<span style="color: #009900;">&#40;</span>e.<span style="color: #006633;">getMessage</span><span style="color: #009900;">&#40;</span><span style="color: #009900;">&#41;</span><span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
  <span style="color: #009900;">&#125;</span>
  emitFooter<span style="color: #009900;">&#40;</span>out<span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
  out.<span style="color: #006633;">close</span><span style="color: #009900;">&#40;</span><span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
<span style="color: #009900;">&#125;</span></pre>
  </div>
</div>

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