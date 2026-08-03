---
title: 'Hibernate Tools: Tips for Reverse Engineering at the Command Line'
author: mdesjardins
categories:
  - blog
tags:
  - hibernate
  - java
  - jpa
  - programming
---

One of the ancillary projects of the Hibernate framework is the [Hibernate Tools][1] toolset. Using Hibernate Tools, you can automatically generate your mapping files (or, if you prefer, JPA annotations), POJOs, and DDL from your database schema.

I&#8217;ve been enamored with the &#8220;Convention over Configuration&#8221; web frameworks lately (e.g., Grails, Django), and wondered how hard it&#8217;d be to reproduce some of their magic ORM functionality using Hibernate Tools. I&#8217;ve concluded that I&#8217;ve still got a <font style="font-weight: bold;">long</font> ways to go, but that I might have some worthwhile tips to share in the meantime.

My progress has been impeded a bit because the documentation for Hibernate Tools is pretty weak. Hibernate Tools was really generated with Eclipse users in mind. A large portion of the documentation is devoted to explaining how to use the IDE. I&#8217;m not using Eclipse, and I intend to use the tools at the command-line, and command-line use really isn&#8217;t targeted.

<font style="font-weight: bold;">0.) Get the tools and dependencies.</font>  
I&#8217;m still in the dark ages &#8211; I use ant instead of maven for builds, and need to track down jar dependencies the old fashioned way. In addition to the usual Hibernate jars, you&#8217;ll also need [hibernate-tools][2], [freemarker][3], and [jtidy][4].

<font style="font-weight: bold;"><br /> 1.) Setup an Ant task</font>  
This is pretty straightforward, and it is explained fairly well in the documentation. First, define yourself a Hibernate Tools task. Mine looks like this:

```xml
 <taskdef name="hibernatetool" classname="org.hibernate.tool.ant.HibernateToolTask">
  <classpath>
    <fileset refid="hibernate.libs"/>
    <fileset refid="hibernatetools.libs"/>
    <fileset refid="apps.libs"/>
    <fileset refid="compile.libs"/>
    <pathelement path="${build}"/>
    <pathelement path="${basedir}"/>
  </classpath>
</taskdef>
```

Next, create a task that uses our new definition. We&#8217;re going to be creating the mapping files and POJOs in this example. It will look something like this:

```xml
<target name="gen_hibernate" depends="compile">
  <delete dir="${genhbm}" />
  <mkdir dir="${genhbm}" />
  <hibernatetool>
    <jdbcconfiguration
        configurationfile="${src}/hibernate-connection.cfg.xml"
        revengfile="hibernate.reveng.xml"
        packagename="us.mikedesjardins.data"
        detectmanytomany="true"
        detectoptimisticlock="true" />
    <hbm2hbmxml destdir="${genhbm}" />
    <hbm2java destdir="${genhbm}">
      <property key="jdk5" value="true" />
      <property key="ejb3" value="false" />
    </hbm2java>
  </hibernatetool>
</target>
```

You&#8217;ll note that I&#8217;ve created a hibernate configuration file above which only contains connection information. This is because I ran into problems when I tried to use a hibernate configuration with cache configuration elements in it. You&#8217;ll also note a reference to a file called hibernate.reveng.xml. This file controls various aspects of the reverse engineering process. Mine is very simple &#8211; I&#8217;m working with MS SQL Server, and I don&#8217;t want it to include any of the system tables which begin with the prefix sys:

```xml
<hibernate-reverse-engineering>
  <!-- Eliminate system tables  -->
  <table-filter name="sys.*" exclude="true">
</hibernate-reverse-engineering>
```

That should be all that you need to do to get started. When you run this ant target, your domain object POJOs and Mappings should end up in the ${genhbm} directory.

<font style="font-weight: bold;">2.) First Problem &#8211; Table Names</font>  
Let&#8217;s say you&#8217;re in a company that prefixes most of it&#8217;s tables with something silly, like &#8220;tb_&#8221;. In that case, Hibernate tools will happily create all kinds of POJOs for you named TbAccount, TbAddress, etc. This is probably not what you want. Fortunately, Hibernate tools let you provide your own &#8220;Reverse Engineering Strategy&#8221; class to deal with situations just like this.

First, update your build target in Ant to include a reference to your strategy class as an attribute of the jdbcconfiguration element:

```xml
<target name="gen_hibernate" depends="compile">
  <delete dir="${genhbm}" />
  <mkdir dir="${genhbm}" />
  <hibernatetool>
    <jdbcconfiguration
        configurationfile="${src}/hibernate-connection.cfg.xml"
        revengfile="hibernate.reveng.xml"
        packagename="us.mikedesjardins.data"
        detectmanytomany="true"
        detectoptimisticlock="true"
        reversestrategy="us.mikedesjardins.hibernate.CustomReverseEngineeringStrategy"/>
    <hbm2hbmxml destdir="${genhbm}" />
    <hbm2java destdir="${genhbm}">
      <property key="jdk5" value="true" />
      <property key="ejb3" value="false" />
    </hbm2java>
  </hibernatetool>
</target>
```

Next, we&#8217;ll need to create the strategy class. The documentation recommends that you extend the <font style="font-weight: bold;">DelegatingReverseEngineeringStrategy</font> class to do this, like this:

```java
package us.mikedesjardins.hibernate;
 
import org.hibernate.cfg.reveng.DelegatingReverseEngineeringStrategy;
import org.hibernate.cfg.reveng.ReverseEngineeringStrategy;
 
public class CustomReverseEngineeringStrategy extends DelegatingReverseEngineeringStrategy {
  public CustomReverseEngineeringStrategy(ReverseEngineeringStrategy delegate {
    super(delegate);
  }
}
```

Unfortunately, I was unable to find any JavaDoc documentation for the Hibernate Tools classes, so I had to do a bit of trial-and-error. It turns out there is a method called tableToClassName that can be overridden. This class accepts a TableIdentifier as its input parameter, and returns the class name in a String. Thus, to remove the tb_ from the class, we could do something like this:

```java
public String tableToClassName(TableIdentifier tableIdentifier) {
  String packageName = "us.mikedesjardins.data";
  String className = super.tableToClassName(tableIdentifier);
  if (className.startsWith("Tb")) {
    className = className.substring(2);
  }
  return className;
}
```

Compile your CustomReverseEngineeringStrategy class, make sure it&#8217;s on the hibernatetools task definition&#8217;s classpath, and re-run the task. Voila! The Tb&#8217;s are gone!

There&#8217;s a similar method for column names named <font style="font-weight: bold;">columnToPropertyName</font> that handles naming your persisted class&#8217;s member variables. In my current project, the primary key columns are named the same as the table name, with &#8220;_id&#8221; appended to the end. However, in the object model, we like to just name the primary key property &#8220;id&#8221; to simplify the creation of generic DAOs. I use the columnToPropertyName method for this.

<font style="font-weight: bold;">2.) Next Problem &#8211; The Generated POJOs need tweaking.</font>  
Perhaps the code that Hibernate generates is not to your liking. Maybe you work with standards-compliance-nazis who want a copyright header at the top of each class file. Or maybe all of your persisted objects implement the same interface for use with generic DAOs.

Under the covers, Hibernate uses [freemarker][3] to generate POJOs and mapping files. The templates that it uses can be edited to your liking, but doing it is not very straightforward.

First, unzip the hibernate tools jar into a temporary directory. Once unzipped, you&#8217;ll find a directory named pojo. This directory contains all of the templates used for POJO generation. Copy that directory into your build area and name it something like hib_templates.

The hbm2java task is actually just an alias for another hibernate tools target called hbmtemplate. With hbmtemplate, you can configure the location of the source templates. So we&#8217;ll remove the hbm2java task from the gen_hibernate target, and replace it with an equivalent hbmtemplate task:


```xml
<target name="gen_hibernate" depends="compile">
  <delete dir="${genhbm}" />
  <mkdir dir="${genhbm}" />
  <hibernatetool>
    <jdbcconfiguration
        configurationfile="${src}/hibernate-connection.cfg.xml"
        revengfile="hibernate.reveng.xml"
        packagename="us.mikedesjardins.data"
        detectmanytomany="true"
        detectoptimisticlock="true"
        reversestrategy="us.mikedesjardins.hibernate.CustomReverseEngineeringStrategy"/>
     <hbm2hbmxml destdir="${genhbm}" />
     <hbmtemplate templateprefix="pojo/"
              destdir="${genhbm}"
              template="hib_templates/Pojo.ftl"
              filepattern="{package-name}/{class-name}.java">
       <property key="jdk5" value="true">
       <property key="ejb3" value="false">
    </hbmtemplate>
  </hibernatetool>
</target>
```

Now that we&#8217;ve told hibernate tools where our template lives, we can edit it to our liking. For example, if we want to add a copyright notice to the top of each generated POJO, we could do so thusly:

```
//
// Copyright 2008 Big Amalgamated Mega Global Software Corp.  All Rights Reserved.
//
${pojo.getPackageDeclaration()}
// Generated ${date} by Hibernate Tools ${version}
 
<#assign classbody>
<#include "PojoTypeDeclaration.ftl"/>; {
.
.
.
(etc)
```

Now that we&#8217;ve done this, we can create more templates that, e.g., generate empty DAO classes, automatically create derived classes, etc.

Hope that helps!


 [1]: http://www.hibernate.org/255.html
 [2]: http://www.hibernate.org/30.html
 [3]: http://freemarker.sourceforge.net/
 [4]: http://jtidy.sourceforge.net/download.html
