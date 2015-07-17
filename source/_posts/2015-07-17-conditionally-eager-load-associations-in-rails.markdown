---
layout: post
title: "Conditionally Eager-load associations in Rails"
date: 2015-07-17 17:02
comments: true
categories: 
---

_(This post originally appeared on Burnside Digital's blog on July 12, 2013.
Alas, Burnside Digital's blog is no more, so I've reposted it here.)_

We recently had a project for a client where we wanted to eagerly load a 
model’s associations, but only under certain conditions – the associated 
models were rendered from a page fragment that was cached. It turns out this 
is pretty easy to do.

The project that we needed this solution for has a basic CMS. The schema for 
the CMS looks something like this:

<center>
  <img src="/images/cms-schema.png"/>
</center>

In other words, our queries were fraught with opportunities for gobs and gobs 
of dreaded “N+1 queries.” To avoid the n+1 queries, we could simply eagerly 
load by invoking include:

```
class HomeController < ApplicationController
  def index
    @homepage = HomePage.find(params[:id], :include => 
      {:page_section =>
      {:page_section_schedule =>
      {:page_fragment =>
      {:carousel_slide => :image}}}})
  end
end
```

However, there was caching to consider – this data rarely changes (at most, 
daily), so it’s also a great candidate for fragment caching. With a fragment 
cache in place, the page looks a bit like this:

```
<html>
 <head>
  <title>Home Page</title>
 </head>
 <body>
  <h1>Home Page</h1>
  <div>
    <%- cache( "homepage-fragment-#{@homepage.cache_key}" ) do %>
      <%= render "content", :locals => { :homepage => @homepage } %>
    <%- end %>
  </div>
 </body>
</html>
```

With fragment caching, all that laborious eager loading isn’t necessary - the 
associations aren’t needed because the data is already cached in a fragment.

To get around this, we simply perform the eager loading only when we need it, in 
the cached fragment:

```
<html>
 <head>
  <title>Home Page</title>
 </head>
 <body>
  <h1>Home Page</h1>
  <div>
    <%- cache( "homepage-fragment-#{@homepage.updated_at}" ) do %>
      @homepage.do_eager_loading
      <%= render "content", :locals => { :homepage => @homepage } %>
    <%- end %>
  </div>
 </body>
</html>
```

The do_eager_loading method uses Rails’ internal eager loading methods to get 
the work done. It looks like this:

```
class HomePage < ActiveRecord::Base
  has_many :page_section_schedules

  def do_eager_loading
    ActiveRecord::Associations::Preloader.new([self], 
      {:page_sections => 
      {:page_section_schedules => 
      {:page_fragment => 
      {:carousel_slides => :image} }}}).run
  end
end
```

And voila – if the content is cached, no eager loading takes place. If the 
cache isn’t populated or is invalid, we eager load.