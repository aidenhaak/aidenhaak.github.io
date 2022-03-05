---
title: Moving To Hugo
date: 2019-06-29
description: I have just put the finishing touches on this site's second migration -- this time from Jekyll to Hugo. I'd been planning on doing this migration for a while but the lack of SCSS/SASS support was holding me back. Then in August 2019 Hugo 0.43 happened and with it the introduction of Hugo Pipes and SCSS/SASS. So there was nothing holding me back (except for a lack of free time of course).
draft: false
type: post
---

I have just put the finishing touches on this site's second migration -- this time from [Jekyll](http://jekyllrb.com/) to [Hugo](https://gohugo.io/). I'd been planning on doing this migration for a while but the lack of SCSS/SASS support was holding me back. Then in August 2019 [Hugo 0.43 happened](https://gohugo.io/news/0.43-relnotes/) and with it the introduction of Hugo Pipes and SCSS/SASS. So there was nothing holding me back (except for a lack of free time of course).

The main motivation for this change is two-fold: 

* I remove my dependencies on a bunch of random ruby gems
* Having a binary file to do the HTML generation is really nice, no setting up a Ruby or Python environment just make sure the Hugo executable is on the path and it's good to go.

Which means reduced maintenance for keep my theme building and hopefully at least a small increase in my blogging and to start planning yet another migration to the blog platform *du jour* (probably [Gatsby](https://www.gatsbyjs.org/)).