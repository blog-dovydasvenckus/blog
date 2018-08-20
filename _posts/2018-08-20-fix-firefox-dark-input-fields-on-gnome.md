---
layout: post
title:  "Fix Firefox unreadable dark input fields on GNOME"
description: Using dark GTK Firefox renders input fields as dark. In some cases input fields are unreadable. This guide will help you solve this problem by setting Firefox to use a light theme
date:   2018-08-20 23:00:00 +0300
categories: linux
tags: firefox gtk gnome theme
image: /assets/images/common/thumbnails/tux.png
twitter_card.image: /assets/images/common/thumbnails/tux.png
---

When using Firefox on a computer with dark GTK theme, Firefox renders input fields using a dark theme.
In some cases, input in fields are unreadable, because field is dark and the text is black.

Most of the pages work fine, because they have CSS themed form fields, which override the default GTK theme.

### Before:
<img src="/assets/images/2018-08-20-2018-08-20-fix-firefox-dark-input-fields-on-gnome/firefox-form-bad.png">

<br><br>

<img src="/assets/images/2018-08-20-2018-08-20-fix-firefox-dark-input-fields-on-gnome/firefox-form-bad-selected.png">

### After:
<img src="/assets/images/2018-08-20-2018-08-20-fix-firefox-dark-input-fields-on-gnome/firefox-fixed.png">


## Solution
For the last couple of years, I have used a browser plugin to fix field rendering.
But lately, there was a rise in browser plugins which were sold to unethical companies,
which modified plugins to spy on user data or inject custom ads.

For some time I have felt uneasy, because of a rise of such threats.
Recently I have stumbled upon a way how to solve this issue without browser plugin.

1. In Firefox URL bar enter about:config.
2. Create new String value with `widget.content.gtk-theme-override` (Right click > new > String)
3. Set value to `Adwaita:light` (if your distribution does not use GNOME desktop, you should use a name of light GTK theme which is installed on your PC)
4. Restart browser

Enjoy better browsing experience. I hope this post was helpful.

As always feel free to ask questions in the comment section.
