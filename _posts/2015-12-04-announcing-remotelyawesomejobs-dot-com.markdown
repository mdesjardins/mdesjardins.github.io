---
layout: post
title: "Announcing RemotelyAwesomeJobs.com"
date: 2015-12-04 16:37
comments: true
categories: 
---
Last October I began working on a site that crawls a bunch of different
job boards looking for remote job posts. I wanted to see if I could go
from literally nothing to a fully-functional, useful site in 30 days
on my own free time.

I decided on this particular project for an odd reason: lots of other
people were building nearly identical sites. You might think that'd be
discouraging, but I decided that proved it'd be pretty trivial to do,
and that there are definitely people out there who would find it useful.
I have no delusions about this site making me rich, so it's no big
deal if there are tons of competitors out there.

In the past I've used project sites like this to learn new technology.
I hemmed and hawed for a long time about whether I wanted to use this
site as a change to learn Elixir+Phoenix and/or ReactJS. In the end I
decided that the challenge was to build something in 30 days, and my
track record with previous "learning" projects were that, while I'd
learn something, they'd almost never get finished. The extra effort
of teaching myself something new made it harder to finish. The goal
for this project was explicitly *not* to learn something new in terms
of technology, it was to see if I really could see a project through
from start to finish given the fewest impedements. It turns out I can!

RemotelyAwesomeJobs is thus unsurprisingly built on Rails 4.2 with
Postgres and Sidekiq in support. There's minimal Javascript (I
decided that might actually be a virtue in this case - I want something
that works on any device with little fuss). I did decide to host it
myself on DigitalOcean instead of going the stupid-easy Heroku
route, mostly because it's a lot cheaper to do so once you want
more than one server.

Anyhow, check it out if you're looking for a remote job. I'm "dog-
fooding" it myself right now, so it should be pretty stable and
usable at this point. ;)

Link:  https://www.remotelyawesomejobs.com
