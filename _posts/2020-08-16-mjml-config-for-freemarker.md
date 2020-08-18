---
layout: post
title: "MJML config for FreeMarker tags"
description: How to configure MJML correctly to work with freemarker tags
date: 2020-08-18 22:30:00 +0300
categories: software-engineering
tags: software-engineering mjml freemarker node js
---

Recently I was working on a project which used `MJML` for building responsive HTML emails and `FreeMarker` as a template engine.
I have faced some hurdles while integrating `MJML` with FreeMarker tags.

## Problem with default config

I have started experiencing issues after enabling CSS inlining.
MJML for CSS inlining uses [Juice](https://github.com/Automattic/juice) library.
Turns out that `Juice` by default doesn't play nicely with FreeMarker tags.

`Juice` by default lower cases all HTML tag attributes, so `<#if task.dateCreated?has_content>` is turned into
`<#if task.datecreated?has_content>`

## Solution

To make MJML play nicely with FreeMarker you must configure `Juice`
to skip processing FreeMarker tags.

You can follow steps below or take a look into sample project
on [GitHub](https://github.com/blog-dovydasvenckus/mjml-freemarker-config).

### Create `.mjmlconfig` config file

```bash
{
  "packages": [],
  "options": {
    "juicePreserveTags": {
      "freeMarkerStartTag": {
        "start": "<#",
        "end": ">"
      },
      "freeMarkerEndTag": {
        "start": "</#",
        "end": ">"
      }
    }
  }
}
```

### Enable config file

While running `mjml` command add --config.useMjmlConfigOptions=true flag, to pull config from `.mjmlconfig` file.

E.g:

```
mjml --config.useMjmlConfigOptions=true -r src/templates/*.mjml -o build/templates/
```

## Conclusion

The solution is pretty simple and straight forward, but it is not well documented.

I hope this article was useful. If you have any questions or suggestions feel free to leave a comment.
