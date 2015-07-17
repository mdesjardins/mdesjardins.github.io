---
title: Why you should not not use ORM
author: mdesjardins
layout: post
permalink: /2008/06/16/why-you-should-not-not-use-orm/
blogger_blog:
  - mikedesjardins.blogspot.com
blogger_author:
  - Mike Desjardinshttp://www.blogger.com/profile/07892630576680251412noreply@blogger.com
blogger_permalink:
  - /2008/06/why-you-should-not-not-use-orm.html
categories:
  - blog
tags:
  - hibernate
  - jpa
  - programming
---
<a onblur="try {parent.deselectBloggerImageGracefully();} catch(e) {}" href="http://mikedesjardins.net/uploaded_images/2522766037_44d8e2e709-731771.jpg"><img style="margin: 0pt 0pt 10px 10px; float: right; cursor: pointer;" src="http://mikedesjardins.net/uploaded_images/2522766037_44d8e2e709-731749.jpg" alt="" border="0" /></a>Yesterday I read a blog post by Kenneth Downs entitled &#8220;[Why I Do Not Use ORM][1]&#8221; on his [The Database Programmer][2] blog. It wasn&#8217;t the first blog post with gripes about Object Relational Mapping, and it certainly won&#8217;t be the last. For me, this particular article highlighted a few misconceptions about why and how ORM should be used, and I thought I might chime in with my own perspective.

<span style="font-weight: bold;">ORM is not a way to avoid SQL</span>  
The first thing that stood out for me was this quote:  
> &#8220;The language SQL is the most widely supported, implemented, and used way to connect to databases. But since most of us have long lists of complaints about the language, we end up writing abstraction layers that make it easier for us to avoid coding SQL directly.&#8221;</p>
To his credit, the author wasn&#8217;t actually addressing ORM in this paragraph. However, anyone writing business logic which interacts with a database, who is unable to write some basic SQL, is like a bull in a china shop; nothing good will come of it regardless of the tools and abstraction layers employed.

<span style="font-weight: bold;">The Simple Example</span>  
The next example in the post shows how one would write a row to a database in only four lines of PHP code &#8211; it looks something like this (I removed the comments):

<div class="wp_syntax">
  <div class="code">
    <pre class="php" style="font-family:monospace;"><span style="color: #000088;">$row</span> <span style="color: #339933;">=</span> getPostStartingWith<span style="color: #009900;">&#40;</span><span style="color: #0000ff;">"inp_"</span><span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
<span style="color: #000088;">$table_id</span> <span style="color: #339933;">=</span> myGetPostVarFunction<span style="color: #009900;">&#40;</span><span style="color: #0000ff;">'table_id'</span><span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
<span style="color: #b1b100;">if</span> <span style="color: #009900;">&#40;</span><span style="color: #339933;">!</span>SQLX_Insert<span style="color: #009900;">&#40;</span><span style="color: #000088;">$table_id</span><span style="color: #339933;">,</span><span style="color: #000088;">$row</span><span style="color: #009900;">&#41;</span><span style="color: #009900;">&#41;</span> <span style="color: #009900;">&#123;</span>
 myFrameworkErrorReporting<span style="color: #009900;">&#40;</span><span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
<span style="color: #009900;">&#125;</span></pre>
  </div>
</div>

I don&#8217;t know PHP, but I think the code is reading every field in a posted form that starts with inp_, generates an insert statement straight from the ID&#8217;s on the form fields, and writes an insert statement with the results.

Perhaps it&#8217;s unfair to criticize this code because &#8220;it&#8217;s just a simple example,&#8221; but if this code is being held up as an example of how short and simple non-ORM code can be, one does have to wonder 
*   When the database schema changes, does the HTML need to be updated so that the form fields match the database schema?
*   Where are the transactions? What if I need to insert into several tables and roll back the transaction if one fails?
*   Does the handy SQLX_insert method prevent SQL injection attacks?

He goes on to say that this task is made even easier by [using a data dictionary][3] to generate the SQL. After reading the &#8220;Using a Data Dictionary&#8221; article, one has to wonder whether or not the author realizes that it is a very crude form of ORM.

<span style="font-weight: bold;">What about Business Logic?</span>  
Kenneth Downs tries to head-off any arguments about business logic before they come up, knowing that ORM evangelists will argue that the domain objects can encapsulate essential business logic for the application. His response?  
> &#8220;The SQLX_Insert() routine can call out to functions (fast) or objects (much slower) that massage data before and after the insert operation. I will be demonstrating some of these techniques in future essays, but of course the best permforming and safest method is to use triggers.&#8221;</p>
For me, this sounds alarm bells. Triggers slow down transactions. Triggers are in your database, which is often your system&#8217;s biggest bottleneck. Triggers silently do things behind your back without telling you. Triggers change databases from vast, efficient places to store relational data, into a lumbering behemoth interpreting procedural code inside big iron.

Conversely, business logic that can be easily distributed across many smaller web servers scales horizontally. The domain layer is a fantastic place to embed simple data massaging &#8211; sadly, I often see a pile of persistent entities with getters and setters that don&#8217;t do anything.

<span style="font-weight: bold;">Counter Example: The Disaster Scenario</span>  
Lastly, Ken (can I call him that? What is the ettiquite for this sort of thing, anyway?) shows an example of a piece of code that is likely to cause hundreds of unneeded reads to the database in an untuned ORM-based system. I don&#8217;t dispute this; in fact, I [posted][4] about an almost identical nightmare scenario myself a while ago.

For this, I go back to my &#8220;bull in a china shop&#8221; analogy. Programmers can write horrible code in any language, with any tool. Layers of abstraction are a double-edged sword, because you need to understand what they are doing for you. But it&#8217;s not the tool&#8217;s fault; it&#8217;s the person misusing it.

<span style="font-weight: bold;">Computer Science&#8217;s Vietnam</span>  
In 2004, Ted Neward famously called ORM [Computer Science&#8217;s Vietnam][5]. Encouraged by early successes, we got sucked into the quagmire. There are plenty of reasons to be frustrated with ORM, but I&#8217;m not sure I agree with Ken&#8217;s. I try to hit the 80/20 rule with ORM, and use it where it makes sense. When I get into a convoluted transaction or need to do a large batch of operations, I&#8217;m not afraid to dive into SQL and do the work in a stored procedure. I think it&#8217;s a good mix. How about you?

<span style="font-style: italic;">Photo Credit: </span><a style="font-style: italic;" href="http://www.flickr.com/people/meesterdickey/">Ryan Dickey</a>

 [1]: http://database-programmer.blogspot.com/2008/06/why-i-do-not-use-orm.html
 [2]: http://database-programmer.blogspot.com/
 [3]: http://database-programmer.blogspot.com/2008/06/using-data-dictionary.html
 [4]: http://mikedesjardins.net/blog/2008/03/pizza-shop-2-totaling-jpa-order-use.html
 [5]: http://blogs.tedneward.com/2006/06/26/The+Vietnam+Of+Computer+Science.aspx