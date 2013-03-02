---
layout: post
title: "Introducing RackSeo"
comments: true
category: 
tags: []
---

I remember reading [How to level up as a
developer](http://jasonrudolph.com/blog/2011/08/09/programming-achievements-how-to-level-up-as-a-developer/)
and thinking that I should try and write a few different types of
programs to challenge myself. I'd been curious about Rack Middleware so
I made it a resolution for 2013 to code up an idea I'd had for a while.

It's a common scenario that website owners to commission a site, then
run an SEO campaign 6 months after the site has launched when they realise they
have no traffic. This often leads to a request
from the SEO people is to sort out unique and relevant meta tags for
each page, matching the text of the h1 element and so on. If this
wasn't considered from the start it can be hard to drop in on some
sites, particulary where there static/hard coded pages involved.

## ...Enter RackSeo 
It's a plain rack middleware which should work with any
Rack based framework or app (Rails, Sinatra, Padrino, Merb etc.).

It processes the final HTML response, sets up the meta tags and
populates them with sensible content based on the text in the page
(using the summarize gem). You
can specify a format for the title tag based on css selectors (eg pull
the h1 text and append the name of the company) and it will work
dynamically on every page _without_ _changing_ _any_ _code_. 

It's configurable using a YAML file and can be set for different paths
including wildcards. 

You can read all about it over on the [Github
readme](https://github.com/xavriley/rack-seo). I'm all ears
on how to take it forward to being a useful tool in the future.

Enjoy!
