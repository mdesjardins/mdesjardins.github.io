---
layout: post
title: "My first npm project: react-a11y-select"
date: 2017-12-08 08:59
comments: true
categories: react, a11y, accessibility, web development
---
I've had a interest/passion/fascination with web accessibility (a11y) ever since I worked at my previous employer, [Sigient](https://www.sigient.com) (note: they might be out of business now, and that site appears to be down). That particular job was a consulting company, and I ended up on the staff augmentation side of the business, which wasn't particularly awesome, but I'll save that story for another time.

I was working with one of the major nationwide internet service providers in the United States. They too are no longer in business, as the corporation was recently acquired. One of the things I undertook was converting some major swaths of their customer portal, an old Backbone web application, to use React. While I working on that task, the company had come under pressure to quickly make their customer portal accessible to users with disabilities. I don't know where this pressure was coming from - there were rumors of a lawsuit, but this was hearsay and I have no idea if there was any truth to it.

At that point in my career, I had very little exposure to web accessibility. In retrospect, it's shameful that this is the case, but I don't believe it's uncommon. I've been developing websites for well over 15 years, and I had never encountered a situation where anyone made accessibility a priority. Because of that, I knew almost nothing about roles or aria attributes, [high contrast color palettes](http://colorsafe.co/), or the imporatance of using semantic elements and design to ensure that screen readers would be able to properly interpret pages. I'm embarrassed to admit that my first exposure to aria attributes was probably seeing them on [Bootstrap](http://getbootstrap.com/) documentation, and thinking to myself "what the heck are those things?"

Since my experience at Sigient, I've made a conscious effort to apply accessibility to my designs whenever I've been afforded the opportunity. Admittedly, my blog needs more contrast and better semantic elements. Ugh, I'll add it to my to-do list!

As I've transitioned to doing a lot more front-end development, particularly in [React](https://reactjs.org/), it has occurred to me that we have an opportunity to reduce the difficulty in building accessible rich internet applications. Because React (and many other newer Javascript frameworks) are component-based, component creators can focus on making their individual components accessible to disabled users, while application developers can use off-the-shelf components which have been tested and meet minimum accessibility guidelines.

![React Logo](https://upload.wikimedia.org/wikipedia/commons/thumb/a/a7/React-icon.svg/1200px-React-icon.svg.png)

This obviously requires buy-in from component developers, but I'm optimistic that component based UI architecture makes this possible and achievable.

## react-a11y-select
To that end, I've started a project on npm to build a web-accessible select component. The first rule of accessibility is to use native components whenever possible. However, over the course of my career I've seen many designs requiring select/dropdown components with styled options containing graphics and other elements. With this project I'm hoping to make doing so in an accessible way super-simple.

I just started this project recently, and it doesn't even meet all the accessibility guidelines yet. It's not as performant as I'd like and some of it is pretty hacky. It's missing some key features that I desperately want to add. But it's a start and it's my very first npm project, and first component library. I hope to keep plugging away at it, and eventually introduce a few more components as I get better at this (I'm thinking I might do Radio buttons next!). 

Here's a link to the project on [Github](https://github.com/mdesjardins/react-a11y-select) and on [npm](https://www.npmjs.com/package/react-a11y-select). There's a [live demo page](http://mikedesjardins.net/react-a11y-select/) too.

Feel free to submit pull requests. :)