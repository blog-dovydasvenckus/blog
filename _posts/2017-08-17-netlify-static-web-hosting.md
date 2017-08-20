---
layout: post
title:  "Netlify static web site hosting"
date:   2017-08-17 22:52:15 +0300
categories: web-hosting
---

## Introduction
Static site generators are really good for blogging sites. You generate site once
and you simply serve generated HTML files using HTTP server.

There are at least a few dozens of static site generators. For this blog I have chosen
to use Jekyll, because it is quite popular and well documented.

## Why should you consider static web sites
Static site load times are fast, because you don't waste time to render web page for every request.
Also by using static site, you totally skip other layers like database.

For some people static blog might look too static, because you can't add dynamic content.
The most common example is comments. The easiest way to solve this problem is to
use a 3rd party comment library. E.g [Disqus](https://disqus.com/). Of couse,if you don't
like 3rd party libraries for privacy reasons you could develop REST API for comment
management and implement comment rendering in JavaScript.

Another thing I love about static web sites is versioning. Because your content
is just a bunch of markup files you can easily version your website using
your favorite versioning system e.g. git, mercurial.

Deployment process is a breeze when you are using git or other versioning system
that allow to execute commands after you push code to the repository. After pushing
your code to the remote, it triggers a command to generate static site and after a short
period of time your changes are live.

## Why I have chose netlify as my blog hosting?
[Netlify](https://netlify.com) CDN. Netlify hosts your website in multiple server that
are located in different geographical locations. This minimizes load times for users,
because content is a served from server that is nearest to their geographical position.

Netlify is easily configurable you can select which branch you want to deploy on
git push. You have access system ENV variables. Also, it should support wide variety
of site generators, because you have access to change the command that builds your site.

On top of easy deployment you get ability to rollback your site to previous version
using web interface.

It even has prerendering option for single page apps. It should help Google web
crawlers to scan your dynamic site.

One click SSL certificate configuration using Let's encrypt. I'm a hude fan of
[Let's encrypt].

And all these features are accessible for free.

## Final thoughts
I only scratched the surface of features that Netlify provides. But I really like
the simplicity and ease of use of their service.
