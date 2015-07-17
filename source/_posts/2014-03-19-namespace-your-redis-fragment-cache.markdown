---
layout: post
title: Namespace Your Redis Fragment Cache
date: 2014-03-19 15:04
comments: true
categories: 
---
For a recent work project at [Openbay](http://www.openbay.com/), we decided to use Resque for our
delayed job processing (might move to Sidekiq eventually, but it's not a priority right now). That
meant we had a Redis server already up and running in our environment. When it came time to implement
partial caching, we knew that memcache would probably have the best performance, but we already
had Redis kicking around anyway, so we figured we'd try using Redis for it instead.

What we *forgot* to do was to namespace our partial cache. This meant that all of our partials were
stored in the same root namespace with everything else, including our delayed jobs.

We eventually noticed that queued jobs were never getting kicked off. While diagnosing the problem,
I decided I'd try clearing the cache (for unrelated reasons). Imagine my horror:

```
irb(main):005:0> Resque.delayed_queue_peek(0,1000)
=> [1393632000, 1393635600, 1393639200, 1393696996, 1393696998, 1393697211, 1393697222, 1393699884, 1393718400, 1393783396, 1393783398, 1393783611, 1393783622, 1393786284, 1393869796, 1393869798, 1393870011, 1393870022, 1393956196, 1393956198, 1393956411, 1393956422, 1393959084, 1394042596, 1394042598, 1394042811, 1394042822, 1394045484, 1394128996, 1394128998, 1394129211, 1394129222, 1394131884, 1394330596, 1394330598, 1394330811, 1394330822, 1394333484, 1394733796, 1394733798, 1394734011, 1394734023, 1394736684, 1395050596, 1395050598, 1395050811, 1395050823, 1395053484]
irb(main):006:0> Rails.cache.clear
=> "OK"
irb(main):007:0> Resque.delayed_queue_peek(0,1000)
=> []
irb(main):008:0>
```

**ZOMG.** Rails.cache.clear was wiping out all of our delayed jobs!

The fix was simple enough - namespace your cache entries in your environment.rb file:

```
  config.cache_store = :redis_store, ENV['REDIS_URL'], { :namespace => 'production-cache' }
```

Don't forget to namespace! After that, we were all better:

```
=> [1393617439, 1393632000, 1393635600, 1393703632, 1393718400, 1393790032, 1393962832, 1394049232, 1394135632, 1394337232, 1394740432, 1395057232]
irb(main):006:0> Rails.cache.clear
=> 20
irb(main):007:0> Resque.delayed_queue_peek(0,1000)
=> [1393617439, 1393632000, 1393635600, 1393703632, 1393718400, 1393790032, 1393962832, 1394049232, 1394135632, 1394337232, 1394740432, 1395057232]
```
