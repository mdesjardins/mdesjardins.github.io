---
title: Last Day of the Month w/ JavaScript
author: mdesjardins
layout: post
permalink: /2007/11/13/last-day-of-month-w-javascript/
blogger_blog:
  - mikedesjardins.blogspot.com
blogger_permalink:
  - /2007_11_01_archive.html
categories:
  - blog
tags:
  - javascript
---
I recently had to calculate the last day of the month using JavaScript. When I Google&#8217;d for a way to do this (I had an idea for a solution already, but I wanted to look online first), I was pretty surprised by how much work some of the proposed solutions were, like [this][1] and (even worse) [this][2]. One trick I learned from SQL programming is that it doesn&#8217;t need to be that complicated. All you need to do is jump to the next month, go to the first day of that month, then subtract one day. Here&#8217;s the code:

``` javascript
now.setMonth(now.getMonth() + 1); // rounds back to January if needed (e.g., 13th month).
now.setDate(0)  // now is now set to the last day of the previous month,
                // i.e., the "zeroth" day of the current month = the last day of the previous month.
```

 [1]: http://javascript.about.com/library/bllday.htm
 [2]: http://www.irt.org/script/1568.htm