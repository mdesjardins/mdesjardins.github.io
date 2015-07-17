---
title: 'Hibernate Tools: Tips for Reverse Engineering at the Command Line'
author: mdesjardins
layout: post
permalink: /2008/08/05/hibernate-tools-tips-for-reverse/
blogger_blog:
  - mikedesjardins.blogspot.com
blogger_permalink:
  - /2008_08_01_archive.html
categories:
  - blog
tags:
  - hibernate
  - java
  - jpa
  - programming
---
<a onblur="try {parent.deselectBloggerImageGracefully();} catch(e) {}" href="http://mikedesjardins.net/uploaded_images/tools-geishaboy500-716138.jpg"><img style="margin: 0pt 0pt 10px 10px; float: right; cursor: pointer;" src="http://mikedesjardins.net/uploaded_images/tools-geishaboy500-716103.jpg" alt="" border="0" /></a>One of the ancillary projects of the Hibernate framework is the [Hibernate Tools][1] toolset. Using Hibernate Tools, you can automatically generate your mapping files (or, if you prefer, JPA annotations), POJOs, and DDL from your database schema.

I&#8217;ve been enamored with the &#8220;Convention over Configuration&#8221; web frameworks lately (e.g., Grails, Django), and wondered how hard it&#8217;d be to reproduce some of their magic ORM functionality using Hibernate Tools. I&#8217;ve concluded that I&#8217;ve still got a <font style="font-weight: bold;">long</font> ways to go, but that I might have some worthwhile tips to share in the meantime.

My progress has been impeded a bit because the documentation for Hibernate Tools is pretty weak. Hibernate Tools was really generated with Eclipse users in mind. A large portion of the documentation is devoted to explaining how to use the IDE. I&#8217;m not using Eclipse, and I intend to use the tools at the command-line, and command-line use really isn&#8217;t targeted.

<font style="font-weight: bold;">0.) Get the tools and dependencies.</font>  
I&#8217;m still in the dark ages &#8211; I use ant instead of maven for builds, and need to track down jar dependencies the old fashioned way. In addition to the usual Hibernate jars, you&#8217;ll also need [hibernate-tools][2], [freemarker][3], and [jtidy][4].

<font style="font-weight: bold;"><br /> 1.) Setup an Ant task</font>  
This is pretty straightforward, and it is explained fairly well in the documentation. First, define yourself a Hibernate Tools task. Mine looks like this:

<div class="wp_syntax">
  <div class="code">
    <pre class="xml" style="font-family:monospace;"><span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;taskdef</span> <span style="color: #000066;">name</span>=<span style="color: #ff0000;">"hibernatetool"</span> <span style="color: #000066;">classname</span>=<span style="color: #ff0000;">"org.hibernate.tool.ant.HibernateToolTask"</span><span style="color: #000000; font-weight: bold;">&gt;</span></span>
  <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;classpath<span style="color: #000000; font-weight: bold;">&gt;</span></span></span>
    <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;fileset</span> <span style="color: #000066;">refid</span>=<span style="color: #ff0000;">"hibernate.libs"</span> <span style="color: #000000; font-weight: bold;">/&gt;</span></span>
    <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;fileset</span> <span style="color: #000066;">refid</span>=<span style="color: #ff0000;">"hibernatetools.libs"</span> <span style="color: #000000; font-weight: bold;">/&gt;</span></span>
    <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;fileset</span> <span style="color: #000066;">refid</span>=<span style="color: #ff0000;">"app.libs"</span> <span style="color: #000000; font-weight: bold;">/&gt;</span></span>
    <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;fileset</span> <span style="color: #000066;">refid</span>=<span style="color: #ff0000;">"compile.libs"</span> <span style="color: #000000; font-weight: bold;">/&gt;</span></span>
    <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;pathelement</span> <span style="color: #000066;">path</span>=<span style="color: #ff0000;">"${build}"</span> <span style="color: #000000; font-weight: bold;">/&gt;</span></span>
    <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;pathelement</span> <span style="color: #000066;">path</span>=<span style="color: #ff0000;">"${basedir}"</span> <span style="color: #000000; font-weight: bold;">/&gt;</span></span>
  <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;/classpath<span style="color: #000000; font-weight: bold;">&gt;</span></span></span>
<span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;/taskdef<span style="color: #000000; font-weight: bold;">&gt;</span></span></span></pre>
  </div>
</div>

Next, create a task that uses our new definition. We&#8217;re going to be creating the mapping files and POJOs in this example. It will look something like this:

<div class="wp_syntax">
  <div class="code">
    <pre class="xml" style="font-family:monospace;"><span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;target</span> <span style="color: #000066;">name</span>=<span style="color: #ff0000;">"gen_hibernate"</span> <span style="color: #000066;">depends</span>=<span style="color: #ff0000;">"compile"</span><span style="color: #000000; font-weight: bold;">&gt;</span></span>
  <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;delete</span> <span style="color: #000066;">dir</span>=<span style="color: #ff0000;">"${genhbm}"</span> <span style="color: #000000; font-weight: bold;">/&gt;</span></span>
  <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;mkdir</span> <span style="color: #000066;">dir</span>=<span style="color: #ff0000;">"${genhbm}"</span> <span style="color: #000000; font-weight: bold;">/&gt;</span></span>
  <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;hibernatetool<span style="color: #000000; font-weight: bold;">&gt;</span></span></span>
    <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;jdbcconfiguration</span></span>
<span style="color: #009900;">        <span style="color: #000066;">configurationfile</span>=<span style="color: #ff0000;">"${src}/hibernate-connection.cfg.xml"</span></span>
<span style="color: #009900;">        <span style="color: #000066;">revengfile</span>=<span style="color: #ff0000;">"hibernate.reveng.xml"</span></span>
<span style="color: #009900;">        <span style="color: #000066;">packagename</span>=<span style="color: #ff0000;">"us.mikedesjardins.data"</span></span>
<span style="color: #009900;">        <span style="color: #000066;">detectmanytomany</span>=<span style="color: #ff0000;">"true"</span></span>
<span style="color: #009900;">        <span style="color: #000066;">detectoptimisticlock</span>=<span style="color: #ff0000;">"true"</span> <span style="color: #000000; font-weight: bold;">/&gt;</span></span>
    <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;hbm2hbmxml</span> <span style="color: #000066;">destdir</span>=<span style="color: #ff0000;">"${genhbm}"</span> <span style="color: #000000; font-weight: bold;">/&gt;</span></span>
    <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;hbm2java</span> <span style="color: #000066;">destdir</span>=<span style="color: #ff0000;">"${genhbm}"</span><span style="color: #000000; font-weight: bold;">&gt;</span></span>
      <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;property</span> <span style="color: #000066;">key</span>=<span style="color: #ff0000;">"jdk5"</span> <span style="color: #000066;">value</span>=<span style="color: #ff0000;">"true"</span> <span style="color: #000000; font-weight: bold;">/&gt;</span></span>
      <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;property</span> <span style="color: #000066;">key</span>=<span style="color: #ff0000;">"ejb3"</span> <span style="color: #000066;">value</span>=<span style="color: #ff0000;">"false"</span> <span style="color: #000000; font-weight: bold;">/&gt;</span></span>
    <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;/hbm2java<span style="color: #000000; font-weight: bold;">&gt;</span></span></span>
  <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;/hibernatetool<span style="color: #000000; font-weight: bold;">&gt;</span></span></span>
<span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;/target<span style="color: #000000; font-weight: bold;">&gt;</span></span></span></pre>
  </div>
</div>

You&#8217;ll note that I&#8217;ve created a hibernate configuration file above which only contains connection information. This is because I ran into problems when I tried to use a hibernate configuration with cache configuration elements in it. You&#8217;ll also note a reference to a file called hibernate.reveng.xml. This file controls various aspects of the reverse engineering process. Mine is very simple &#8211; I&#8217;m working with MS SQL Server, and I don&#8217;t want it to include any of the system tables which begin with the prefix sys:

<div class="wp_syntax">
  <div class="code">
    <pre class="xml" style="font-family:monospace;"><span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;hibernate-reverse-engineering<span style="color: #000000; font-weight: bold;">&gt;</span></span></span>
  <span style="color: #808080; font-style: italic;">&lt;!-- Eliminate system tables  --&gt;</span>
  <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;table-filter</span> <span style="color: #000066;">name</span>=<span style="color: #ff0000;">"sys.*"</span> <span style="color: #000066;">exclude</span>=<span style="color: #ff0000;">"true"</span><span style="color: #000000; font-weight: bold;">&gt;</span></span>
<span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;/hibernate-reverse-engineering<span style="color: #000000; font-weight: bold;">&gt;</span></span></span></pre>
  </div>
</div>

That should be all that you need to do to get started. When you run this ant target, your domain object POJOs and Mappings should end up in the ${genhbm} directory.

<font style="font-weight: bold;">2.) First Problem &#8211; Table Names</font>  
Let&#8217;s say you&#8217;re in a company that prefixes most of it&#8217;s tables with something silly, like &#8220;tb_&#8221;. In that case, Hibernate tools will happily create all kinds of POJOs for you named TbAccount, TbAddress, etc. This is probably not what you want. Fortunately, Hibernate tools let you provide your own &#8220;Reverse Engineering Strategy&#8221; class to deal with situations just like this.

First, update your build target in Ant to include a reference to your strategy class as an attribute of the jdbcconfiguration element:

<div class="wp_syntax">
  <div class="code">
    <pre class="xml" style="font-family:monospace;"><span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;target</span> <span style="color: #000066;">name</span>=<span style="color: #ff0000;">"gen_hibernate"</span> <span style="color: #000066;">depends</span>=<span style="color: #ff0000;">"compile"</span><span style="color: #000000; font-weight: bold;">&gt;</span></span>
  <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;delete</span> <span style="color: #000066;">dir</span>=<span style="color: #ff0000;">"${genhbm}"</span> <span style="color: #000000; font-weight: bold;">/&gt;</span></span>
  <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;mkdir</span> <span style="color: #000066;">dir</span>=<span style="color: #ff0000;">"${genhbm}"</span> <span style="color: #000000; font-weight: bold;">/&gt;</span></span>
  <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;hibernatetool<span style="color: #000000; font-weight: bold;">&gt;</span></span></span>
    <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;jdbcconfiguration</span></span>
<span style="color: #009900;">        <span style="color: #000066;">configurationfile</span>=<span style="color: #ff0000;">"${src}/hibernate-connection.cfg.xml"</span></span>
<span style="color: #009900;">        <span style="color: #000066;">revengfile</span>=<span style="color: #ff0000;">"hibernate.reveng.xml"</span></span>
<span style="color: #009900;">        <span style="color: #000066;">packagename</span>=<span style="color: #ff0000;">"us.mikedesjardins.data"</span></span>
<span style="color: #009900;">        <span style="color: #000066;">detectmanytomany</span>=<span style="color: #ff0000;">"true"</span></span>
<span style="color: #009900;">        <span style="color: #000066;">detectoptimisticlock</span>=<span style="color: #ff0000;">"true"</span></span>
<span style="color: #009900;">        <span style="color: #000066;">reversestrategy</span>=<span style="color: #ff0000;">"us.mikedesjardins.hibernate.CustomReverseEngineeringStrategy"</span><span style="color: #000000; font-weight: bold;">/&gt;</span></span>
    <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;hbm2hbmxml</span> <span style="color: #000066;">destdir</span>=<span style="color: #ff0000;">"${genhbm}"</span> <span style="color: #000000; font-weight: bold;">/&gt;</span></span>
    <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;hbm2java</span> <span style="color: #000066;">destdir</span>=<span style="color: #ff0000;">"${genhbm}"</span><span style="color: #000000; font-weight: bold;">&gt;</span></span>
      <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;property</span> <span style="color: #000066;">key</span>=<span style="color: #ff0000;">"jdk5"</span> <span style="color: #000066;">value</span>=<span style="color: #ff0000;">"true"</span> <span style="color: #000000; font-weight: bold;">/&gt;</span></span>
      <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;property</span> <span style="color: #000066;">key</span>=<span style="color: #ff0000;">"ejb3"</span> <span style="color: #000066;">value</span>=<span style="color: #ff0000;">"false"</span> <span style="color: #000000; font-weight: bold;">/&gt;</span></span>
    <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;/hbm2java<span style="color: #000000; font-weight: bold;">&gt;</span></span></span>
  <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;/hibernatetool<span style="color: #000000; font-weight: bold;">&gt;</span></span></span>
<span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;/target<span style="color: #000000; font-weight: bold;">&gt;</span></span></span></pre>
  </div>
</div>

Next, we&#8217;ll need to create the strategy class. The documentation recommends that you extend the <font style="font-weight: bold;">DelegatingReverseEngineeringStrategy</font> class to do this, like this:

<div class="wp_syntax">
  <div class="code">
    <pre class="java" style="font-family:monospace;"><span style="color: #000000; font-weight: bold;">package</span> <span style="color: #006699;">us.mikedesjardins.hibernate</span><span style="color: #339933;">;</span>
&nbsp;
<span style="color: #000000; font-weight: bold;">import</span> <span style="color: #006699;">org.hibernate.cfg.reveng.DelegatingReverseEngineeringStrategy</span><span style="color: #339933;">;</span>
<span style="color: #000000; font-weight: bold;">import</span> <span style="color: #006699;">org.hibernate.cfg.reveng.ReverseEngineeringStrategy</span><span style="color: #339933;">;</span>
&nbsp;
<span style="color: #000000; font-weight: bold;">public</span> <span style="color: #000000; font-weight: bold;">class</span> CustomReverseEngineeringStrategy <span style="color: #000000; font-weight: bold;">extends</span> DelegatingReverseEngineeringStrategy <span style="color: #009900;">&#123;</span>
  <span style="color: #000000; font-weight: bold;">public</span> CustomReverseEngineeringStrategy<span style="color: #009900;">&#40;</span>ReverseEngineeringStrategy delegate <span style="color: #009900;">&#123;</span>
    <span style="color: #000000; font-weight: bold;">super</span><span style="color: #009900;">&#40;</span>delegate<span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
  <span style="color: #009900;">&#125;</span>
<span style="color: #009900;">&#125;</span></pre>
  </div>
</div>

Unfortunately, I was unable to find any JavaDoc documentation for the Hibernate Tools classes, so I had to do a bit of trial-and-error. It turns out there is a method called tableToClassName that can be overridden. This class accepts a TableIdentifier as its input parameter, and returns the class name in a String. Thus, to remove the tb_ from the class, we could do something like this:

<div class="wp_syntax">
  <div class="code">
    <pre class="java" style="font-family:monospace;"><span style="color: #000000; font-weight: bold;">public</span> <span style="color: #003399;">String</span> tableToClassName<span style="color: #009900;">&#40;</span>TableIdentifier tableIdentifier<span style="color: #009900;">&#41;</span> <span style="color: #009900;">&#123;</span>
  <span style="color: #003399;">String</span> packageName <span style="color: #339933;">=</span> <span style="color: #0000ff;">"us.mikedesjardins.data"</span><span style="color: #339933;">;</span>
  <span style="color: #003399;">String</span> className <span style="color: #339933;">=</span> <span style="color: #000000; font-weight: bold;">super</span>.<span style="color: #006633;">tableToClassName</span><span style="color: #009900;">&#40;</span>tableIdentifier<span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
  <span style="color: #000000; font-weight: bold;">if</span> <span style="color: #009900;">&#40;</span>className.<span style="color: #006633;">startsWith</span><span style="color: #009900;">&#40;</span><span style="color: #0000ff;">"Tb"</span><span style="color: #009900;">&#41;</span><span style="color: #009900;">&#41;</span> <span style="color: #009900;">&#123;</span>
    className <span style="color: #339933;">=</span> className.<span style="color: #006633;">substring</span><span style="color: #009900;">&#40;</span><span style="color: #cc66cc;">2</span><span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
  <span style="color: #009900;">&#125;</span>
  <span style="color: #000000; font-weight: bold;">return</span> className<span style="color: #339933;">;</span>
<span style="color: #009900;">&#125;</span></pre>
  </div>
</div>

Compile your CustomReverseEngineeringStrategy class, make sure it&#8217;s on the hibernatetools task definition&#8217;s classpath, and re-run the task. Voila! The Tb&#8217;s are gone!

There&#8217;s a similar method for column names named <font style="font-weight: bold;">columnToPropertyName</font> that handles naming your persisted class&#8217;s member variables. In my current project, the primary key columns are named the same as the table name, with &#8220;_id&#8221; appended to the end. However, in the object model, we like to just name the primary key property &#8220;id&#8221; to simplify the creation of generic DAOs. I use the columnToPropertyName method for this.

<font style="font-weight: bold;">2.) Next Problem &#8211; The Generated POJOs need tweaking.</font>  
Perhaps the code that Hibernate generates is not to your liking. Maybe you work with standards-compliance-nazis who want a copyright header at the top of each class file. Or maybe all of your persisted objects implement the same interface for use with generic DAOs.

Under the covers, Hibernate uses [freemarker][3] to generate POJOs and mapping files. The templates that it uses can be edited to your liking, but doing it is not very straightforward.

First, unzip the hibernate tools jar into a temporary directory. Once unzipped, you&#8217;ll find a directory named pojo. This directory contains all of the templates used for POJO generation. Copy that directory into your build area and name it something like hib_templates.

The hbm2java task is actually just an alias for another hibernate tools target called hbmtemplate. With hbmtemplate, you can configure the location of the source templates. So we&#8217;ll remove the hbm2java task from the gen_hibernate target, and replace it with an equivalent hbmtemplate task:

<div class="wp_syntax">
  <div class="code">
    <pre class="xml" style="font-family:monospace;"><span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;target</span> <span style="color: #000066;">name</span>=<span style="color: #ff0000;">"gen_hibernate"</span> <span style="color: #000066;">depends</span>=<span style="color: #ff0000;">"compile"</span><span style="color: #000000; font-weight: bold;">&gt;</span></span>
  <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;delete</span> <span style="color: #000066;">dir</span>=<span style="color: #ff0000;">"${genhbm}"</span> <span style="color: #000000; font-weight: bold;">/&gt;</span></span>
  <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;mkdir</span> <span style="color: #000066;">dir</span>=<span style="color: #ff0000;">"${genhbm}"</span> <span style="color: #000000; font-weight: bold;">/&gt;</span></span>
  <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;hibernatetool<span style="color: #000000; font-weight: bold;">&gt;</span></span></span>
    <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;jdbcconfiguration</span></span>
<span style="color: #009900;">        <span style="color: #000066;">configurationfile</span>=<span style="color: #ff0000;">"${src}/hibernate-connection.cfg.xml"</span></span>
<span style="color: #009900;">        <span style="color: #000066;">revengfile</span>=<span style="color: #ff0000;">"hibernate.reveng.xml"</span></span>
<span style="color: #009900;">        <span style="color: #000066;">packagename</span>=<span style="color: #ff0000;">"us.mikedesjardins.data"</span></span>
<span style="color: #009900;">        <span style="color: #000066;">detectmanytomany</span>=<span style="color: #ff0000;">"true"</span></span>
<span style="color: #009900;">        <span style="color: #000066;">detectoptimisticlock</span>=<span style="color: #ff0000;">"true"</span></span>
<span style="color: #009900;">        <span style="color: #000066;">reversestrategy</span>=<span style="color: #ff0000;">"us.mikedesjardins.hibernate.CustomReverseEngineeringStrategy"</span><span style="color: #000000; font-weight: bold;">/&gt;</span></span>
     <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;hbm2hbmxml</span> <span style="color: #000066;">destdir</span>=<span style="color: #ff0000;">"${genhbm}"</span> <span style="color: #000000; font-weight: bold;">/&gt;</span></span>
     <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;hbmtemplate</span> <span style="color: #000066;">templateprefix</span>=<span style="color: #ff0000;">"pojo/"</span></span>
<span style="color: #009900;">              <span style="color: #000066;">destdir</span>=<span style="color: #ff0000;">"${genhbm}"</span></span>
<span style="color: #009900;">              <span style="color: #000066;">template</span>=<span style="color: #ff0000;">"hib_templates/Pojo.ftl"</span></span>
<span style="color: #009900;">              <span style="color: #000066;">filepattern</span>=<span style="color: #ff0000;">"{package-name}/{class-name}.java"</span><span style="color: #000000; font-weight: bold;">&gt;</span></span>
       <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;property</span> <span style="color: #000066;">key</span>=<span style="color: #ff0000;">"jdk5"</span> <span style="color: #000066;">value</span>=<span style="color: #ff0000;">"true"</span><span style="color: #000000; font-weight: bold;">&gt;</span></span>
       <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;property</span> <span style="color: #000066;">key</span>=<span style="color: #ff0000;">"ejb3"</span> <span style="color: #000066;">value</span>=<span style="color: #ff0000;">"false"</span><span style="color: #000000; font-weight: bold;">&gt;</span></span>
    <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;/hbmtemplate<span style="color: #000000; font-weight: bold;">&gt;</span></span></span>
  <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;/hibernatetool<span style="color: #000000; font-weight: bold;">&gt;</span></span></span>
<span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;/target<span style="color: #000000; font-weight: bold;">&gt;</span></span></span></pre>
  </div>
</div>

Now that we&#8217;ve told hibernate tools where our template lives, we can edit it to our liking. For example, if we want to add a copyright notice to the top of each generated POJO, we could do so thusly:

<div class="wp_syntax">
  <div class="code">
    <pre class="java" style="font-family:monospace;"><span style="color: #666666; font-style: italic;">//</span>
<span style="color: #666666; font-style: italic;">// Copyright 2008 Big Amalgamated Mega Global Software Corp.  All Rights Reserved.</span>
<span style="color: #666666; font-style: italic;">//</span>
$<span style="color: #009900;">&#123;</span>pojo.<span style="color: #006633;">getPackageDeclaration</span><span style="color: #009900;">&#40;</span><span style="color: #009900;">&#41;</span><span style="color: #009900;">&#125;</span>
<span style="color: #666666; font-style: italic;">// Generated ${date} by Hibernate Tools ${version}</span>
&nbsp;
<span style="color: #339933;">&lt;</span>#assign classbody<span style="color: #339933;">&gt;</span>
<span style="color: #339933;">&lt;</span>#include <span style="color: #0000ff;">"PojoTypeDeclaration.ftl"</span><span style="color: #339933;">/&gt;;</span> <span style="color: #009900;">&#123;</span>
.
.
.
<span style="color: #009900;">&#40;</span>etc<span style="color: #009900;">&#41;</span></pre>
  </div>
</div>

Now that we&#8217;ve done this, we can create more templates that, e.g., generate empty DAO classes, automatically create derived classes, etc.

Hope that helps!

<font style="font-style: italic;">Photo Credit: </font><a style="font-style: italic;" href="http://flickr.com/people/geishaboy500/">geishaboy500</a>

 [1]: http://www.hibernate.org/255.html
 [2]: http://www.hibernate.org/30.html
 [3]: http://freemarker.sourceforge.net/
 [4]: http://jtidy.sourceforge.net/download.html