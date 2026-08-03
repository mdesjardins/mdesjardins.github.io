---
title: Use Hibernate Validators to find Nasty DataTruncation exceptions
author: mdesjardins
layout: post
permalink: /2008/06/10/use-hibernate-validators-to-find-nasty/
blogger_blog:
  - mikedesjardins.blogspot.com
blogger_author:
  - Mike Desjardinshttp://www.blogger.com/profile/07892630576680251412noreply@blogger.com
blogger_permalink:
  - /2008/06/use-hibernatevalidators-to-find-nasty.html
categories:
  - blog
tags:
  - hibernate
  - java
  - jpa
---
If you develop Hibernate applications against either Sybase or MS SQL Server databases, you may have had the pleasure of seeing this little gem:

```
09:13:22,155 WARN [] org.hibernate.util.JDBCExceptionReporter – SQL Error: 0, SQLState: 22001
09:13:22,155 ERROR [] org.hibernate.util.JDBCExceptionReporter – Data truncation
09:13:22,155 WARN [] org.hibernate.util.JDBCExceptionReporter – SQL Error: 8152, SQLState: 22001
09:13:22,155 ERROR [] org.hibernate.util.JDBCExceptionReporter – String or binary data would be truncated.
09:13:22,159 ERROR [] org.hibernate.event.def.AbstractFlushingEventListener – Could not synchronize database state with session
.
.
.
Caused by: java.sql.DataTruncation: Data truncation
```

The most maddening thing about these errors is that the database won&#8217;t tell you <span style="font-style: italic;">which</span> column is being 
truncated. A likely scenario is that you&#8217;ve developed a Web or Swing UI that permits the user to enter a value for a String field which is 
ultimately persisted to a database column, and the UI is not restricting the maximum length of the user&#8217;s input to the size of that the column 
supports. The question is, which field is the offender?

<span style="font-weight: bold;">Hibernate Validators to the Rescue!</span>  
Situations like this are why it&#8217;s a pretty good idea to validate your data at the domain layer of your project, and not rely on your database 
and presentation layer to do all of the validation. Fortunately, Hibernate makes this pretty painless with validator annotations. Just import the 
org.hibernate.validator.Length annotation, and add the @Length annotation in your entity class, e.g.:

``` java
@Column(name="description",length=20)
@Length(max=20)
private String description;
```

Note that putting the length attribute in the @Column annotation <span style="font-style: italic;">is not enough</span>. That attribute is used 
for generating DDL, but will not cause any validation to take place.

Now that our new Annotation is in place, we get the <span style="font-style: italic;">slightly</span> more useful stack-trace below:

```
org.hibernate.validator.InvalidStateException: validation failed for: us.mikedesjardins.data.persist.MyClass
at org.hibernate.validator.event.ValidateEventListener.validate(ValidateEventListener.java:148)
at org.hibernate.validator.event.ValidateEventListener.onPreInsert(ValidateEventListener.java:172)
at org.hibernate.action.EntityIdentityInsertAction.preInsert(EntityIdentityInsertAction.java:119)
.
.
.
```

Of course, you should catch these exceptions and do something less hostile with them. For now, we&#8217;ll just log the details and re-throw so 
you can get the general gist of what you can do. Here&#8217;s a generic insert method in a DAO base class:

``` java
public void insert(T valueObject, Class... clazz) {
 try {
   Session s = getSession();
   s.save(valueObject);
   s.flush();
   s.refresh(valueObject);
 } catch (InvalidStateException v) {
   setToRollback();
   log.error("insert(), failed validation " + valueObject.toString(), v);
   InvalidValue[] invalid = v.getInvalidValues();
   for (int i=; i<invalid.length; ++i) {
     InvalidValue bad = invalid[i];
     log.error("insert(), " + bad.getPropertyPath()
     + ":" + bad.getPropertyName()
     + ":" + bad.getMessage());
   }
   throw v;
 }
}
```


As you can see, the InvalidStateException contains an array of InvalidValue objects which describe the nature of the validation 
error. You can either log this information, or even present it to the user.

Hope that helps! Do you use Hibernate&#8217;s validator annotations for something nifty? Let us know in the comments!