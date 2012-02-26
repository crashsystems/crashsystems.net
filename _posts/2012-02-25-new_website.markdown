---
layout: post
title: Crashsystems.net 2.0
aside: Creating a better website with Jekyll, Foundation CSS and Nginx
tags:
- Jekyll
- Foundation CSS
- Nginx
---

As I've not updated this site in well over a year, and do not feel particularly attached to any of my old content, I recently decided to delete the old site and start from scratch. Also, I've gotten tired of always updating my WordPress installation. This, along with the fact that I'm sufficiently paranoid that I worry about having a database and server-side code running on my VPS, seemed like the perfect excuse to play with some new tools that I've been meaning to try. In this post I'll outline the tools I used to create crashsystems.net 2.0.

###Jekyll
The best way to describe [Jekyll](http://jekyllrb.com/) is that it is like WordPress themes without the the rest of WordPress, and without having to bother with writing PHP. Templates are created with the [Liquid](http://liquidmarkup.org/) templating language, and content is written with [Markdown](http://daringfireball.net/projects/markdown/), a simple system for expressing content formatting in plaintext. Jekyll converts the Markdown content to HTML, runs the Liquid templates to assemble the site, and then outputs static HTML.

This setup has several advantages, the first being security. Since I'm not running a database server or any server-side scripting language on this site, there is much less that an attacker could attack. Secondly, no database queries means my site loads a lot faster. Theoretically, my site performance is only limited by the web server I'm running (more on that in a bit), disk I/O and connection throughput. Thirdly, my content is plain text stored in files, so I can edit it in any program I want, and I don't have to worry about database backups.

Comments don't work with static files, so I'm outsourcing comments to [Disqus](http://disqus.com/). Also, for site search I'm using a bit of JavaScript to redirect users to a Google search using the _site:_ parameter.

###Foundation
[Foundation](http://localhost:4000/2012/02/25/new_website/) is a CSS framework for creating [responsive](http://www.alistapart.com/articles/responsive-web-design/) website layouts, meaning that one site template can automatically adjust to varying device form factors. Basically, I created this layout for desktops, and got a mobile site automatically, with no extra effort. In addition to simplifying the development of layouts, Foundation also abstracts many of the tedious aspects of using CSS. I'm far from being a graphics designer, but with Foundation I was able to quickly construct a site design that I don't think looks terrible.

###Nginx
Apache is a very versatile web server, but it also seems somewhat slow and resource intensive. After I did some research, I found that the [Nginx](http://wiki.nginx.org/Main) web server seems to be one of the fastest servers, especially for serving static content. The Nginx installation process was quite simple, and [this](http://blog.martinfjordvald.com/2011/04/optimizing-nginx-for-high-traffic-loads/) guide was quite useful for performance tuning the server.

---

The source code for my site can be found on [GitHub](https://github.com/crashsystems/crashsystems.net). Now that the site is set up, I plan on posting somewhat more frequently, about technology, information security, crypto and perhaps a bit of photography.