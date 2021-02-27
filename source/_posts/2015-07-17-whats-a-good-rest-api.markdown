---
layout: post
title: "What's a (Good) REST API?"
date: 2015-07-17 16:40
comments: true
categories: 
---

_(This post originally appeared on Burnside Digital's blog on July 12, 2013.
Alas, Burnside Digital's blog is no more, so I've reposted it here.)_
 
Recently, a colleague who is a front-end developer asked me what the qualities 
were for a good REST API. He had been experimenting with 
[AngularJS](http://angularjs.org) and wanted 
to get a rough idea of whether or not he could expect the resource objects 
returned by Angular’s $resource factory to work nicely with them.

Strangely, this was a question hard to pin down an answer for. I’ve worked with 
RESTful APIs for years and had a good idea what made a good REST API (“I know 
one when I see it”), but never really given much thought to it. 
[Roy Fielding](http://web.archive.org/web/20140707035207/http://en.wikipedia.org/wiki/Roy_Fielding)
first defined REST in his 
[doctoral dissertation](http://web.archive.org/web/20140707035207/http://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm)
back in 2000, but that tome was a lot thicker than the quick bullet-points my 
colleague was looking for.

RESTfulness is something that happens in degrees. I think I might classify it 
thusly:

### Not RESTful, even though the API creators might claim that it’s RESTful These alone don’t make an API RESTful:

* Uses HTTP or HTTPS
* Returns XML and/or JSON

Occasionally you’ll run into some misguided folks who think that the above two 
points are all it takes to slap the REST seal-of-approval on their APIs. 
Ironically, REST pedants will tell you that specifying the protocol or format 
not only is inadequate qualification for RESTfulness, but it actually runs 
afoul of the spirit of REST.

Warning signs that a self-proclaimed REST interface that isn’t:

* Verbs are in your URL (e.g., http://example.com/reservation/get or http://example.com/business/search)
* You are using POSTDATA to send query parameters even though you aren’t changing the server’s application state.

### Mostly RESTful
A mostly (and I’d claim adequately) RESTful interface has these qualities:

* Uses URLs to refer to resources. These resources are the “nouns” in the 
system. Examples of a resource might be an account, a transaction, a user, 
an appointment, etc.
* Uses the four best-known, well-recognized “HTTP Verbs”, and defines them 
semantically to mean the following “CRUD” operations:
 - GET – read a resource or list of resources
 - POST – create a new resource
 - PUT – modify an existing resource
 - DELETE – delete a resource
* You might wonder how you actually *do* something like, e.g., a payment, in a 
system designed so noun-centrically like this. It seems actions/verbs have no 
place in it. However, even in these cases you are creating a transaction. So 
you’d POST/create a payment resource. Everything is expressed in terms of 
resource lifecycle events.
* Uses semantically appropriate error codes. E.g., 404 means the resource isn’t 
found. A 401 means you aren’t authorized to view it. A 422 means something 
about the data or request was malformed.

### Hardcore RESTful / Hypermedia
The highest degree of RESTful APIs I’ll refer to as “Hardcore RESTful.” They 
have the following qualities:

* Doesn’t necessarily define a protocol/transport (e.g., HTTP). As I said 
above, pedantic REST nerds will tell you that restricting REST to a single 
protocol violates the spirit of REST. The reality is that REST almost always 
implies HTTP.
* Doesn’t define something like a version number in the URL – a version isn’t 
semantically a resource or a part of the resource. Instead API versioning can be 
done using something like HTTP header values (e.g., Accept:), or some other 
side-band portion of the protocol.
* Similarly, it shouldn’t define a desired data format in the URL 
(e.g., http://example.com/foo.json), and should instead do that via some 
side-band part of the protocol like HTTP headers (e.g., content Accept:). You 
can embed versioning and desired format in a MIME type that is passed as an 
Accept header like this: *application/vnd.mycompany.myapp-v1+json*
* To take a RESTful interface up to the next level, you can create an API that 
builds upon REST and does not return hierarchical data. Instead it returns URLs 
to subsets of data. This style of API is usually referred to as a “Hypermedia 
API” for its reliance on hyperlinks. E.g.,

#### BAD
```
<account>
  <name>Checking</name>
  <balance>100.00</balance>
  <user>
    <name>Elvis Presley</name>
    <address>Graceland</address>
    <city>Memphis</city>
    <state>TN</state>
  </user>
</account>
```

#### GOOD
```
<account>
  <name>Checking</name>
  <balance>100.00</balance>
  <user href="http://example.com/user/elvis-presley"/>
</account>
```
The argument for this style of design are that your inner URLs can change at a 
later date, and not break the entire API. It also enables “discovery” of the 
interface by allowing a client to start at a root node of the API, following 
links much like a search-engine spider or bot would. This also engenders systems 
to evolve their APIs independently -theoretically, a client with a generic 
ability to interpret hypermedia shouldn’t get tripped up by a less rigid 
definition of the API. Proponents of Hypermedia APIs often speak of a concept 
called 
[“Hypermedia as the Engine of Application State“](http://web.archive.org/web/20140707035207/http://en.wikipedia.org/wiki/HATEOAS), 
or HATEOAS for short.

Arguments against this style is that, in practice, this manner of API 
discoverability is of limited usefulness and actually incurs an overhead by 
requiring more API calls than would be otherwise needed.

### In Summary

What I told my colleague was this: most REST APIs out there meet the “mostly 
RESTful” level of compliance. Given that, most REST client abstractions will 
probably work OK if the API is at least at that degree of RESTfulness (with the 
caveat that I didn’t wade very deeply into Angular’s resource library!). If 
you’re creating a brand new REST API, it’s good to aim a little higher than 
that - you’re clients will appreciate it.

If you want to read more about REST and Hypermedia APIs, Steve Klabnik has 
written an 
[excellent](http://web.archive.org/web/20140707035207/http://blog.steveklabnik.com/posts/2011-07-03-nobody-understands-rest-or-http)
[series](http://web.archive.org/web/20140707035207/http://blog.steveklabnik.com/posts/2011-08-07-some-people-understand-rest-and-http) of 
[blog](http://web.archive.org/web/20140707035207/http://blog.steveklabnik.com/posts/2012-02-23-rest-is-over) 
[posts](http://web.archive.org/web/20140707035207/http://blog.steveklabnik.com/posts/2012-02-13-an-api-ontology)  
which expand on these ideas a whole lot more.