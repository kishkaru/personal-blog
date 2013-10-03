---
layout: post
title: "Hello Blog"
date: 2013-10-02 19:57
comments: true
categories: 
---

So after beating around the bush for about two months now, I've finally decided to set up myself a blog.

The choice wasn't all that easy. I've thought about Wordpress; it's straightforward to set up and the
user interface is dead simple for creating blog posts and administration. But all that simplicity comes
at a price: performance. Wordpress works by having a SQL database in the back to store all the posts, 
using PHP as glue code to generate the dynamic front end on the fly when a user hits the site. Sure, there
are ways to optimize Wordpress to run faster like setting up caches and such, but all these take time
and I wasn't so ingrained in Wordpress to spend all the time to do that. Ain't no free lunch.

So a dear friend of mine recommended [Octopress](http://octopress.org). It's a blogging framework that
uses [Jekyll](https://github.com/mojombo/jekyll) and Ruby underneath to generate static web files for hosting. 
No running PHP conversions on the fly, and for web hosting, it can't git simpler than hosting static HTML. 
I simply create a new post in [Markdown](http://en.wikipedia.org/wiki/Markdown) and Octopress does the rest by 
a one-time convert from Markdown into HTML with ```rake generate```. Since I'm hosting all this myself, I can 
easily SSH into my box to create new posts and push any changes to GitHub as backup. Easy right?

I've also considered some other static file generators like [Nikola](http://getnikola.com/) and 
[Pelican](http://blog.getpelican.com/), but these are Python-based tools and I'm more familiar with Ruby. Plus,
for a class project this semester, I'll be using Ruby and Rails extensively so as I learn new tricks, I can extend
my blog with new goodies or poke around to see out how everything works underneath. Exciting!