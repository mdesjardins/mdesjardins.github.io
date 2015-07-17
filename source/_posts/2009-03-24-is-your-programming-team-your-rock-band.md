---
title: Is Your Programming Team Your Rock Band?
author: mdesjardins
layout: post
permalink: /2009/03/24/is-your-programming-team-your-rock-band/
categories:
  - blog
tags:
  - programming
---
I&#8217;m sure that I&#8217;m not the person to make this connection, but it occurred to me the other day that being on a small
 team of coders is a lot like being in a band.  I&#8217;ve been in a couple bands that never went far beyond the garage (I&#8217;m allegedly a bass player), so perhaps I&#8217;m not the foremost authority on this topic.  However, I think there are a few parallels between building, e.g., a small MVC web application, and writing and performing the next standard verse-chorus-verse rock anthem.

<img class="alignright size-medium wp-image-235" title="tereu-tereu-ian-matthew-soper" src="http://www.cereslogic.com/pages/wp-content/uploads/2009/03/tereu-tereu-ian-matthew-soper-300x200.jpg" alt="tereu-tereu-ian-matthew-soper" width="300" height="200" />

In particular, I think there are parallels between the specific members of a prototypical Rock band, and the members of a team who create MVC applications:

*   **Drummer** &#8211; The Anchor.  Provides the foundation for the music, onto which the other layers are stacked and woven.  In an MVC app, this is your database guy/gal.  The person modeling your data and managing the schematic underpinnings of your application is your drummer.  And if you&#8217;re using a weird schema-less database like CouchDB, then you have yourself a sloppy jazz drummer, which means more work for the Bassist.
*   **Bass Player** &#8211; The Bass Player sets the groove for the song, and maps the primal beats that the drummer is hammering out into something melodic for the rest of the band to work with.  The bassist is also crucial for carrying the beat for the rest of the band when the drummer is screwing around (see note about jazz drummers, above).  In our web application, this is the domain layer, where your ORM, caching, and validation all chill out.  And like the bass player, this is rarely the sexiest or most glamorous part of the application.
*   **Guitarist(s) **- This is your business logic developer.  Some bands choose to break this up into your traditional Angus/Malcom roles of &#8220;rhythm&#8221; and &#8220;lead&#8221; guitar.  Likewise, you may choose to have separate model and controller layers in your app (particularly if you&#8217;re a three-tier app: the web tier controller is your Angus and the middle tier model is your Malcom).  Bass players can sometimes get by as guitarists in a pinch, and vice versa.  Likewise, you&#8217;ll see business logic guys doing domain work and vice versa.  Just don&#8217;t let them try to play their parts on the wrong instrument &#8211; you&#8217;ve gotta separate your concerns, dude.  Aside from the lyrics, this is the part of the song that your listeners are the most likely to hear.
*   **Lead Singer** &#8211; This is your AJAXy, CSS laden, poetic presentation of your band&#8217;s message.  It&#8217;s what your listeners hear (and see) first.  And the singer gets all flustered when the rest of the band screws up (the singer is a real primadonna, very sensitive). Changing your lead singer will almost certainly alieniate your fans (think Facebook redesign &#8211; the Sammy Hagar of UI decisions).

I tend to get carried away with analogies, and this one is no different &#8211; I could keep going (your roadies are your project managers, your label is the marketing and executives who make all the money) but I&#8217;ll try to show some restraint.  But given this analogy, I find it interesting that the role I play on software teams is often similar to the role I play in a band.  I like bass.

*Photo Credit: [Ian Matthew Soper][1]*

 [1]: http://www.flickr.com/people/iansoper/