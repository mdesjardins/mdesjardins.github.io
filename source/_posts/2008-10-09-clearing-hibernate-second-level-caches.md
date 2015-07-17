---
title: Clearing Hibernate Second-Level Caches
author: mdesjardins
layout: post
permalink: /2008/10/09/clearing-hibernate-second-level-caches/
blogger_blog:
  - mikedesjardins.blogspot.com
blogger_permalink:
  - /2008_10_01_archive.html
categories:
  - blog
tags:
  - cache
  - hibernate
  - java
  - jpa
---
Recently, I wanted to be able to clear out all of the Hibernate caches via JBoss&#8217;s JMX console. I could have taken the easy way out; we&#8217;re using EHCache, so it could have been as simple as calling <span style="font-style: italic;">CacheManager.clearAll()</span>. However, that would have tied me to a specific cache provider. We&#8217;re still evaluating switching to other cache providers. Ideally, my solution would not be dependent on a specific cache implementation.

Hibernate&#8217;s API does provide a simple way to clear specific caches, but does not provide any method for clearing out all of them. Writing your own is fairly straightforward. First, you obtain all of the entity and collection metadata from the session factory. Next you iterate over the entities, and if the object is cached, you clear out all of the caches associated with the persisted class or collection. Here&#8217;s the code:

``` java
  @PersistenceContext
  private EntityManager em;
 
  public void clearHibernateCache() {
    Session s = (Session)em.getDelegate();
    SessionFactory sf = s.getSessionFactory();
    Map<string,EntityPersister> classMetadata = sf.getAllClassMetadata();
 
    for (EntityPersister ep : classMetadata.values()) {
      if (ep.hasCache()) {
        sf.evictEntity(ep.getCache().getRegionName());
      }
    }
 
    Map<string,AbstractCollectionPersister> collMetadata = sf.getAllCollectionMetadata();
    for (AbstractCollectionPersister acp : collMetadata.values()) {
      if (acp.hasCache()) {
        sf.evictCollection(acp.getCache().getRegionName());
      }
    }
    return;
  }
```

Now, if we decide to switch to a different cache provider, this code will not need to be re-written. Hopefully we won&#8217;t ever change to a different JPA implementaion. :)
