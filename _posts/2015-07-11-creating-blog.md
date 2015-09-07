---
layout: post
title: "Creating a Blog with Github-Pages and Jekyll"
published: true
categories:
  - Tutorial
tags:
  - "github-pages"
  - jekyll
  - blog
permalink: /creating-blog/
---



After a few moment looking at some solutions (jekyll, jekyll-Bootstrap), I found a very easy and quick one named [Jekyll-Now](http://www.jekyllnow.com/). It's especially detailed in this [blog post from SmashingMagazine](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet).

I won't repeat this post because it's exactly what you need to create an easy blog hosted on Github for free.

**Some details :**

- It's free
- You just have to fork a repository and you can start writing your articles
- Everything can be written in the browser
- You use only markdown ...
- ... but you can add HTML if you want
- don't need to install anything

##### DISQUS

I did have some problems configuring the disqus (disqus error).

My fix :

 - remove `tdeheurles.com` and `blog.tdeheurles.com` => just redirect to `tdeheurles.github.io` now
 - removed solution from jekyllnow and use the code proposed by disqus :
   - https://publishers.disqus.com/engage?utm_source=Home-Nav
   - follow the setup with `tdeheurles.github.io`
   - setup `tdeheurles.github.io` in `CNAME` and `config.yml`
