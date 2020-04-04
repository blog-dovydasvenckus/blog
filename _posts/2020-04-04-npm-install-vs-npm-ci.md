---
layout: post
title: "NPM ci vs install"
description: NPM ci and install command differences
date: 2020-04-04 19:30:00 +0300
categories: javascript
tags: node npm javascript
---

While working with multiple people on the same project,
I have always been annoyed by the way `npm install` works.
Running `npm install` after checking out code always caused
different `package-lock.json` than `package-lock.json` checked in source control.

Also, there is a bigger problem when `npm install` is used on CI server.
CI server might update `package-lock.json` and install newer
versions of packages than intended. It might lead to bugs that
were not present on the developer's machine.

`npm install` had `--no-save` flag which allowed to install packages without
modifying `package-lock.json`. So there was a workaround, but it was
not that convenient. Simple human factor error could cause, CI server to build a broken app.

## NPM ci to the rescue

In the spring of 2018 **Node 6.0** was released, which contained new `npm ci` command.

Main differences from `npm install`:

- Removes `node_modules` directory and installs new `node_modules` using `package-lock.json`.
- Never updates `package.json` or `package-lock.json`. If packages in `package.json` do not match with packages in `package-lock.json` it will throw error.
- Can't install new packages.

While working locally if someone updated `packages.json` I can pull code and run `npm ci`,
without generating different `package-lock.json`.
`npm ci` does not replace `npm install`, because it can't update `package-lock.json`.
I use `npm install` to sync `package-lock.json` with changed `packages.json`.

## Conclusion

I'm glad that Node team included `npm ci` command.
It's quite useful locally and when running node builds on automated build servers.
It's a pity that I found this command not that long ago.
