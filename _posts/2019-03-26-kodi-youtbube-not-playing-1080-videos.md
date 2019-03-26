---
layout: post
title:  "Kodi Youtube plugin capped at 720p video"
description: Kodi Youtube max video resolution 720p on Raspberry PI
date:   2019-03-26 23:35:00 +0300
categories: kodi
tags: kodi raspberry
image: /assets/images/common/thumbnails/kodi.png
twitter_card.image: /assets/images/common/thumbnails/kodi.png
---

Lately I have been playing around with Kodi on Rasberry Pi 2. 

And I have noticed that all of Youtube videos seem pixelated on my 1080p monitor.
After poking around I have found out, that videos are being played at 720p resolution.
<img src="/assets/images/2019-03-26-kodi-youtube-720p/before.png">

After digging around online, I have found a solution for this problem.

### Solution
In this article I'll be using `Kodi 18.0`

1. Install `InputStream Adaptive addon`

2. Open Youtube `Add-on` settings. (To open sidebar you can press left arrow button, when in Youtube addon)
<img src="/assets/images/2019-03-26-kodi-youtube-720p/youtube-settings-menu.png">

3. Navigate to MPEG-DASH and enable `Use MPEG-DASH` toggle
<img src="/assets/images/2019-03-26-kodi-youtube-720p/input-stream-config.png">

4. Then go into `Configure InputStream Adaptive` and set `Min.Bandwidth (Bit/s)` to arbritary big value such as `10000000`.
<img src="/assets/images/2019-03-26-kodi-youtube-720p/min-bandwidth.png">

5. Save changes and restart your device. 

After these steps videos should be playing at 1080 or higher resolution.
<img src="/assets/images/2019-03-26-kodi-youtube-720p/after.png">

As always if you have any questions or suggestion, feel free to comment below.