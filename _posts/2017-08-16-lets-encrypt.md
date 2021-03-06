---
layout: post
title:  "Let's encrypt is awesome. Free SSL for everyone!"
description: Let's encrypt making internet more secure by providing free SSL certificates.
date:   2017-08-16 22:48:42 +0300
categories: security
tags: security HTTS SSL
---

<img src="/assets/images/2017-08-16-lets-encrypt/firefox-no-cert.png">

I'm sure that we all have seen this window more than once. After you created an app
with your sweat and tears it's time to deploy it. And to avoid this ugly window
you will need to acquire SSL certificate that web browser will trust.

In my experience SSL certificates were a pain in the ass. You had to acquire them by buying,
and you had to configure your HTTP server to use the certificate.

## Let's encrypt - the game changer
[Let's encrypt](https://letsencrypt.org/) is certificate authority. It is a nonprofit organization that provides
*domain validation certificates* for free.

Even better, it provides free SSL certificates in a fully automated way. It uses
[ACME](https://github.com/ietf-wg-acme/acme/) protocol. To acquire certificate
you need to use ACME client. For this, you can use official Let's encrypt client
[certbot](https://certbot.eff.org/).

On top of that, all **Let's encrypt** embraces open source mindset and publishes
their code on GitHub. They even have open sourced their ACME [server](https://github.com/letsencrypt/boulder).

Also, Let's encrypt have partnered up with some hosting providers to provide
their SSL certificates by default. One of the providers that I personally use is
[netlify](https://www.netlify.com)

## Acquiring certificate using certbot
Certbot has automatized certificate issuance and configuration for most popular HTTP servers.
It includes Apache, Nginx support.

For example, to acquire SSL certificate and configure HTTPS on Nginx server using
Arch Linux I ran these commands:

    sudo pacman -S certbot-nginx
    sudo certbot --nginx

The first command installs certbot.

The last command launches an interactive installer that asks a few questions before
acquiring SSL certificate.

That's it, it's stupidly simple. After restart Nginx should use new SSL certificate.
I recommend to reviewing config files to make sure it configured it correctly.

## Renewing certificate using certbot
Let's encrypt certificates are valid only for 90 days. But it is really easy
to auto-renew this certificate.

    certbot renew

This command renews all certificates that were issued using certbot.

To automate this process, you can add this command to cron.

My crontab:

    0 0 3 * * ? /usr/bin/certbot renew --quiet

This cron job will be run every day at 3 AM.



## Some caveats
I'm not saying that Let's encrypt made other certificate providers obsolete.
Let's encrypt only provides *domain validation certificates*. They do not provide
*organization validation certificates* and other types, because they can't automate
issuing of these certificates.

Certificates are valid for only 90 days. But as I have written it is easy to renew
them automatically.

Currently, Let's encrypt does not support wildcard certificates. But they have
[promised](https://letsencrypt.org/2017/07/06/wildcard-certificates-coming-jan-2018.html)
to add support in January of 2018.

Older Java versions did not trust Let's encrypt certificates by [default](https://community.letsencrypt.org/t/will-the-cross-root-cover-trust-by-the-default-list-in-the-jdk-jre/134/60).
But it was fixed in Java 7 and Java 8 versions. So if you are running latest revision of
these versions you should be fine.

Automatic certificate renewing does not work when your server is behind [CloudFlare](https://www.cloudflare.com/)
or other similar services. In that case, renewing certificate involves manual work.
[This is great article](http://infosec.theos-blog.com/renewing-letsencrypt-certificate-when-using-cloudflare/)
on renewing certificate, when the server is behind [CloudFlare](https://www.cloudflare.com/).

## Final thoughts
I'm really impressed by what Let's encrypt has done. They made HTTPS accessible for everyone.
I believe that free SSL and automated issuance is a huge step toward widespread
HTTPS adoption and safer web.

P.S. From now on this blog uses HTTPS using an SSL certificate provided by Let's encrypt.
I'm really grateful for their service.
