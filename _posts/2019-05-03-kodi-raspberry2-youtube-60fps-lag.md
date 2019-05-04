---
layout: post
title:  "Raspberry PI 2 Kodi Youtube 60 FPS video lag"
description: Fixing Youtube 60 FPS video lag, on raspberry PI 2
date:   2019-05-04 23:30:00 +0300
categories: kodi
tags: kodi libreELEC raspberry
image: /assets/images/common/thumbnails/kodi.png
twitter_card.image: /assets/images/common/thumbnails/kodi.png
---

I was experiencing video lag with **60 FPS** **Youtube** videos on **Raspberry PI 2** running **LibreELEC 9.0**. 60 FPS videos were choppy, video lagged behind audio and was completely unwatchable.

After playing around with overclocking settings I have found out that stock RPI2 GPU clock is just a little bit underpowered for 60FPS video. Small overclock should fix this issue.

By default RPI2 GPU clock speed is 250 MHz. By boosting GPU clock to 300 MHz I have solved Youtube 60 FPS video lag.

The overclocking procedure may be a little different depending on your Raspberry PI OS distribution. Only location of `config.txt` file might be different, but otherwise fix should be the same.


In my case I'm using LibreELEC:

1. SSH into your LibreELEC box.

2. Mount boot partition as Read-Write mode: 

    `mount -o remount,rw /flash`

3. Edit `/flash/config.txt` with vi or nano. E.g. `nano /flash/config.txt`

    Find `force_turbo=0` and set it to `1`. Setting it to `1` will disable dynamic CPU and GPU frequency scaling. With GPU overclock and dynamic scaling I have experienced the same lag.

    Add line `gpu_freq=300` to overclock your GPU.
    
    Your `/flash/config.txt` should look like this:
    ```
    ...
    force_turbo=1
    gpu_freq=300
    ...
    ```

4. Remount boot partition as Read only:

     `mount -o remount,ro /flash`

5. Reboot your system

Because overclocking is small it should not require overvolting. In case you're doing more serious overclock you might need to play around with `over_voltage` option in `config.txt` file. **Overvolt at your own risk, because might lead to hardware failure**.  Read more about overclocking in official RPI [documentation](https://www.raspberrypi.org/documentation/configuration/config-txt/overclocking.md)

That's it Youtube 60 FPS video problem should be solved, by small overclock. You can pat yourself on the back, crack open your favorite drink and enjoy fluid 60 FPS Youtube videos.

As always if you have any questions or suggestion, feel free to comment below. 