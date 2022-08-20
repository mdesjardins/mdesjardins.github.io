---
title: 'Don&#8217;t Ignore serialVersionUID'
author: mdesjardins
layout: post
categories:
  - blog
tags:
  - ejb
  - java
  - jee
  - remoting
---

Okay,I admit that this one should have totally been obvious to me long ago. But I&#8217;m still a bit of a JEE newcomer (been doing it for almost five years), so perhaps I can be forgiven.

If you do a lot of ORM or EJB remoting, you probably deal with a lot of Serializable classes. And you&#8217;re probably used to the annoying warning message that you see all the time in your IDE when you&#8217;re working with Serializable classes:

The serializable class BlaBlaBla does not declare a static final serialVersionUID field of type long BlaBlaBla.java myProject/src/main/java/us/mikedesjardins/foo/domain/entity line 44

If you&#8217;re like me, you roll your eyes and politely add a @SuppressWarnings(&#8220;serial&#8221;) to the top of the class definition (or, worse, you just shut the warning message off in your IDE altogether. Even I don&#8217;t do that!). You reason with yourself that current versions of Java conveniently and automatically compute the serialVersionUID at run-time, so there&#8217;s no need to bother with the formality of a version number on your class &#8211; it&#8217;s just a nuisance holdover from days of Java yore.

<span style="font-weight: bold;">IT&#8217;S A TRAP!</span>  
Now that I&#8217;ve found myself well into a new project with this lazy philosophy, I&#8217;m starting to run into problems. I have a client of my EJB that uses one of these Serializable objects, and I&#8217;m finding that when I make the most trivial changes to my shared classes, I need to compile both the server and the client components. The two components that were supposed to be loosely coupled are now hopelessly intertwined. So I did some further research on how the JVM computes the ad-hoc serialVersionUID at runtime when it isn&#8217;t provided.

[This article over at JavaWorld does a far better and more thorough job of explaining it than I will][1]. In a nutshell, backward-compatability with respect to serialization and de-serialization is a lot less fragile than the cases that the serialVersionUID generation is protecting you against. That version generation algorithm computes an SHA hash based on the class name, sorted member variables, modifiers, and interfaces.

In reality, serialization and de-serialization generally only breaks when one of the following things happens to your class (from the aforementioned article at JavaWorld): 
*   Delete fields
*   Change class hierarchy
*   Change non-static to static
*   Change non-transient to transient
*   Change type of a primitive field

<span style="font-weight: bold;">Ensure Minimal Coupling Between Components</span>  
To ensure that your components which use Serialization have minimal runtime dependencies on each other, you have two options: 
*   Declare a specific serialVersionUID, and update it whenever you make a change that breaks backward compatability.
*   Don&#8217;t rely on any classes for use as transfer objects which will potentially change. This one is pretty obvious, but sometimes you will be surprised down the road at which classes are modified more often than others.
*   Don&#8217;t use your own objects at all when transferring data. Instead, rely on classes like Integers, Strings, or HashMaps to shuttle data around among components. (Obviously, protocols like SOAP and REST rely on XML documents for this to ensure maximum de-coupling, but you&#8217;re presumably using something like EJB remoting to avoid the complexity or overhead of these protocols).

