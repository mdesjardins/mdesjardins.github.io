---
title: How I debugged a JRuby Stack Overflow
author: admin
layout: post
permalink: /2011/03/18/how-i-debugged-a-jruby-stack-overflow/
categories:
  - blog
---
A little over a year ago, I inherited someone else's JRuby on Rails project. My initial role 
on the team was the 'token Java guy,' and knew almost zero about Ruby. I've since grown to love 
JRuby, but that's a story for another time.

Over the course of the project, I've upgraded my JRuby version, upgraded gems, upgraded Java, 
and who knows what else. One day, I went to do a migration, and they just plain didn't work 
anymore. I discovered that most migration-related stuff didn't work, including tasks like 
db:test:load. And by "didn't work anymore," I mean this:

    Meep:~/_work/sonymusic/src/checkout> rake db:test:load
    (in /Users/mdesjardins/_work/sonymusic/src/checkout)
    Invalid access of stack red zone 0x100401b20 rip=0x10a6ceb6e
    Bus error

My initial reaction to this is what I expect most people's reaction would be: "what in holy hell 
is this?" It's basically a blown stack - the stack has "edges" called red zones to which you should 
never write, because it means you're getting too close to going off the end, and the OS is trying 
to prevent you from being stupid.

Google was little help - <a href="http://stackoverflow.com/questions/1501770/invalid-access-of-stack-red-zone-from-java-vm" target="_blank">most people speculated</a> that similar errors were problems w/ OSX's JVM implementation.

So I flailed for a while. I tried backing up to version 1.5 of Apple's JVM (which itself is 
<a href="http://hints.macworld.com/article.php?story=20100123192950640" target="_blank">nontrivial</a>, 
and was made more complicated by the fact that json-jruby requires Java 6, but I digress). When 
that failed, I tried rolling back random gems and versions of JRuby. I tried 
<a href="http://stackoverflow.com/questions/2207233/how-to-enable-full-coredumps-on-os-x" target="_blank">enabling core files</a> 
and reading stacks in gdb, but that ended up being useless. I tried increasing the stack to a whopping 2 Gig 
(normal is 2M) by passing -J-Xss2048M, but it still crashed.

When I passed -J-d32 on JRuby's command line, I got a nice "Stack Limit Exceeded" error instead of the 
nasty crash - so that was a start. Everything pointed to an infinite recursion problem, but where?

I was getting desperate. As it turns out, before I became a retread Ruby programmer, I worked in Java 
enterprise stuff. This is the kind of stuff you do at banks and insurance companies, in hellish beige 
cubicles, where you hang posters with diagrams of middleware and service busses and ORMs.

In beige cubicle hell, I learned about a few tools to monitor active JVMs (this comes in handy when 
you're monitoring god-awful beasts like "enterprise application servers.") I figured it was worth 
a shot to try out **jstack. **jstack comes with the JVM. It shows you a stack dump of all the 
threads running in a JVM. If you're using it on a local JVM, all you need to do is pass it the PID 
of the process you'd like to observe w/ the -l (for local?) command line switch.

The trick was going to be catching it just before it blew up. After a few tries, I caught this:

    "main" prio=5 tid=102801000 nid=0&#215;100601000 runnable [10047d000]   java.lang.Thread.State: RUNNABLE 
    at com.mysql.jdbc.util.ReadAheadInputStream.readFromUnderlyingStreamIfNecessary(ReadAheadInputStream.java:123) 
    at com.mysql.jdbc.util.ReadAheadInputStream.read(ReadAheadInputStream.java:188) - locked <7fd39d7c8> (a com.mysql.jdbc.util.ReadAheadInputStream) 
    at com.mysql.jdbc.MysqlIO.readFully(MysqlIO.java:2329) 
    at com.mysql.jdbc.MysqlIO.reuseAndReadPacket(MysqlIO.java:2774)
    at com.mysql.jdbc.MysqlIO.reuseAndReadPacket(MysqlIO.java:2763)
    at com.mysql.jdbc.MysqlIO.checkErrorPacket(MysqlIO.java:3299)
    at com.mysql.jdbc.MysqlIO.sendCommand(MysqlIO.java:1837)
    at com.mysql.jdbc.MysqlIO.sqlQueryDirect(MysqlIO.java:1961)
    at com.mysql.jdbc.ConnectionImpl.execSQL(ConnectionImpl.java:2537) - locked <7fd3992c0> (a java.lang.Object)
    at com.mysql.jdbc.ConnectionImpl.execSQL(ConnectionImpl.java:2466)
    at com.mysql.jdbc.StatementImpl.executeQuery(StatementImpl.java:1383) - locked <7fd3992c0> (a java.lang.Object)
    at com.mysql.jdbc.DatabaseMetaData$9.forEach(DatabaseMetaData.java:4815)
    at com.mysql.jdbc.IterateBlock.doForAll(IterateBlock.java:50)
    at com.mysql.jdbc.DatabaseMetaData.getTables(DatabaseMetaData.java:4793)
    at arjdbc.jdbc.RubyJdbcConnection$16.call(RubyJdbcConnection.java:1015)
    at arjdbc.jdbc.RubyJdbcConnection.withConnectionAndRetry(RubyJdbcConnection.java:1191)
    at arjdbc.jdbc.RubyJdbcConnection.tables(RubyJdbcConnection.java:576)
    at arjdbc.jdbc.RubyJdbcConnection.tables(RubyJdbcConnection.java:552)
    at arjdbc.jdbc.RubyJdbcConnection$i$tables.call(RubyJdbcConnection$i$tables.gen:65535)
    at org.jruby.runtime.callsite.CachingCallSite.call(CachingCallSite.java:103)
    at rubyjit.tables_AA8BB5A33C226C4FACC724EF93DE8D1DF04792EB.__file__(/Users/mdesjardins/.rvm/gems/jruby-1.6.0@rails238/gems/activerecord-jdbc-adapter-1.1.1/lib/arjdbc/jdbc/adapter.rb:234)
    at rubyjit.tables_AA8BB5A33C226C4FACC724EF93DE8D1DF04792EB.__file__(/Users/mdesjardins/.rvm/gems/jruby-1.6.0@rails238/gems/activerecord-jdbc-adapter-1.1.1/lib/arjdbc/jdbc/adapter.rb)
    at org.jruby.ast.executable.AbstractScript.__file__(AbstractScript.java:37)
    at org.jruby.internal.runtime.methods.JittedMethod.call(JittedMethod.java:127)
    at org.jruby.runtime.callsite.CachingCallSite.call(CachingCallSite.java:103)
    at rubyjit.auto_create_table_E1FB7FE0CE1D294ABA67860B81492049B1D6AE7F.__file__(/Users/mdesjardins/_work/sonymusic/src/checkout/vendor/plugins/auto_migrations/lib/auto_migrations.rb:70)
    at rubyjit.auto_create_table_E1FB7FE0CE1D294ABA67860B81492049B1D6AE7F.__file__(/Users/mdesjardins/_work/sonymusic/src/checkout/vendor/plugins/auto_migrations/lib/auto_migrations.rb)
    at org.jruby.internal.runtime.methods.JittedMethod.call(JittedMethod.java:87)
    at org.jruby.runtime.callsite.CachingCallSite.callBlock(CachingCallSite.java:78)
    at org.jruby.runtime.callsite.CachingCallSite.call(CachingCallSite.java:84)
    at rubyjit.method_missing_with_auto_migration_CA4124448766A2D0B44A8F3F5A4A9EC9908F2D02.__file__(/Users/mdesjardins/_work/sonymusic/src/checkout/vendor/plugins/auto_migrations/lib/auto_migrations.rb:55)
    at rubyjit.method_missing_with_auto_migration_CA4124448766A2D0B44A8F3F5A4A9EC9908F2D02.__file__(/Users/mdesjardins/_work/sonymusic/src/checkout/vendor/plugins/auto_migrations/lib/auto_migrations.rb)
    at org.jruby.internal.runtime.methods.JittedMethod.call(JittedMethod.java:87)
    at org.jruby.internal.runtime.methods.DefaultMethod.call(DefaultMethod.java:148)
    at org.jruby.internal.runtime.methods.AliasMethod.call(AliasMethod.java:101)
    at org.jruby.internal.runtime.methods.AliasMethod.call(AliasMethod.java:101)
    at org.jruby.runtime.callsite.CachingCallSite.callBlock(CachingCallSite.java:78)
    at org.jruby.runtime.callsite.CachingCallSite.call(CachingCallSite.java:84)
    at rubyjit.auto_create_table_E1FB7FE0CE1D294ABA67860B81492049B1D6AE7F.__file__(/Users/mdesjardins/_work/sonymusic/src/checkout/vendor/plugins/auto_migrations/lib/auto_migrations.rb:71)
    at rubyjit.auto_create_table_E1FB7FE0CE1D294ABA67860B81492049B1D6AE7F.__file__(/Users/mdesjardins/_work/sonymusic/src/checkout/vendor/plugins/auto_migrations/lib/auto_migrations.rb)
    at org.jruby.internal.runtime.methods.JittedMethod.call(JittedMethod.java:87)
    at org.jruby.runtime.callsite.CachingCallSite.callBlock(CachingCallSite.java:78)
    at org.jruby.runtime.callsite.CachingCallSite.call(CachingCallSite.java:84)
    at rubyjit.method_missing_with_auto_migration_CA4124448766A2D0B44A8F3F5A4A9EC9908F2D02.__file__(/Users/mdesjardins/_work/sonymusic/src/checkout/vendor/plugins/auto_migrations/lib/auto_migrations.rb:55)
    at rubyjit.method_missing_with_auto_migration_CA4124448766A2D0B44A8F3F5A4A9EC9908F2D02.__file__(/Users/mdesjardins/_work/sonymusic/src/checkout/vendor/plugins/auto_migrations/lib/auto_migrations.rb)
    at org.jruby.internal.runtime.methods.JittedMethod.call(JittedMethod.java:87)
    at org.jruby.internal.runtime.methods.DefaultMethod.call(DefaultMethod.java:148)
    at org.jruby.internal.runtime.methods.AliasMethod.call(AliasMethod.java:101)
    at org.jruby.internal.runtime.methods.AliasMethod.call(AliasMethod.java:101)
    at org.jruby.runtime.callsite.CachingCallSite.callBlock(CachingCallSite.java:78)
    at org.jruby.runtime.callsite.CachingCallSite.call(CachingCallSite.java:84)
    at rubyjit.auto_create_table_E1FB7FE0CE1D294ABA67860B81492049B1D6AE7F.__file__(/Users/mdesjardins/_work/sonymusic/src/checkout/vendor/plugins/auto_migrations/lib/auto_migrations.rb:71)
    at rubyjit.auto_create_table_E1FB7FE0CE1D294ABA67860B81492049B1D6AE7F.__file__(/Users/mdesjardins/_work/sonymusic/src/checkout/vendor/plugins/auto_migrations/lib/auto_migrations.rb)
    at org.jruby.internal.runtime.methods.JittedMethod.call(JittedMethod.java:87)
    at org.jruby.runtime.callsite.CachingCallSite.callBlock(CachingCallSite.java:78)
    at org.jruby.runtime.callsite.CachingCallSite.call(CachingCallSite.java:84)
    at rubyjit.method_missing_with_auto_migration_CA4124448766A2D0B44A8F3F5A4A9EC9908F2D02.__file__(/Users/mdesjardins/_work/sonymusic/src/checkout/vendor/plugins/auto_migrations/lib/auto_migrations.rb:55)
    at rubyjit.method_missing_with_auto_migration_CA4124448766A2D0B44A8F3F5A4A9EC9908F2D02.__file__(/Users/mdesjardins/_work/sonymusic/src/checkout/vendor/plugins/auto_migrations/lib/auto_migrations.rb)
    at org.jruby.internal.runtime.methods.JittedMethod.call(JittedMethod.java:87)
    at org.jruby.internal.runtime.methods.DefaultMethod.call(DefaultMethod.java:148)
    at org.jruby.internal.runtime.methods.AliasMethod.call(AliasMethod.java:101)
    at org.jruby.internal.runtime.methods.AliasMethod.call(AliasMethod.java:101)
    at org.jruby.runtime.callsite.CachingCallSite.callBlock(CachingCallSite.java:78)
    at org.jruby.runtime.callsite.CachingCallSite.call(CachingCallSite.java:84)
    at rubyjit.auto_create_table_E1FB7FE0CE1D294ABA67860B81492049B1D6AE7F.__file__(/Users/mdesjardins/_work/sonymusic/src/checkout/vendor/plugins/auto_migrations/lib/auto_migrations.rb:71)
    at rubyjit.auto_create_table_E1FB7FE0CE1D294ABA67860B81492049B1D6AE7F.__file__(/Users/mdesjardins/_work/sonymusic/src/checkout/vendor/plugins/auto_migrations/lib/auto_migrations.rb)
    at org.jruby.internal.runtime.methods.JittedMethod.call(JittedMethod.java:87)
    at org.jruby.runtime.callsite.CachingCallSite.callBlock(CachingCallSite.java:78)
    at org.jruby.runtime.callsite.CachingCallSite.call(CachingCallSite.java:84)

etc. etc. etc. ad infinitum. This was the "Aha!" moment. It turned out the problem was in 
my vendor/auto_migration plugin. What does this plugin do? I honestly have no idea - from 
the README, it looks like it's a tool that applies schema deltas based on schema.rb instead 
of using migrations the intended way (hmm... that seems... misguided). As I said at the 
start of this post, I inherited this project. I wasn't even using this thing anymore.

I deleted the plugin, and *voila*! My stack trace woes disappeared.

**TL;DR: Use jstack if you're stuck w/ a blown stack on JRuby. It works like a champ.**
