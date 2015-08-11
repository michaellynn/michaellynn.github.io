---
summary: "Make Minecraft (and more) run as-is on OS X without Apple Java"
layout: post
tags: python osx
date: "2015-08-10 18:15:00"
title: Even More Oracle Java Compatibility (Minecraft!)
---

[Rich Trouton](https://twitter.com/rtrouton) kindly wrote [a wonderful blog post](https://derflounder.wordpress.com/2015/08/08/modifying-oracles-java-sdk-to-run-java-applications-on-os-x/) covering [a python script of mine](https://gist.github.com/pudquick/349f063c242239952a2e/64c295fc4576d7df9b8632e6ddecbe6165fd7663) (_note: this is the old version, don't use it!_) which modifies installations of Oracle's Java JDK to extend its capabilities to allow it to run JavaApplicationStub .app bundles as was [discussed](https://forums.developer.apple.com/message/6741) on Apple's Developer forums (go ahead and read it, Apple has made the forum open to the public for now). 

One thing that annoyed me a bit about the fix though was that it didn't seem to work with _everything_ (read: [Minecraft](http://minecraft.net/)), just a large number of Java .apps.

After it was posted, a Twitter friend of mine [asked for a little more help](https://twitter.com/morningwoodspor/status/630886277538258945) with one of these more problematic apps.

I was able to confirm that the app in question could run if I provided the right arguments to ```/usr/bin/java```, so why couldn't JavaApplicationStub launching work?

A little more research (aka Google) turned up [this marvelous additional tidbit](http://apple.stackexchange.com/a/136976).

Basically the problematic Java .apps in question had their .jar files coded to look for ```libserver.dylib```, which isn't part of the current JDK distribution.

This fix involves creating the original directory structure it expects it at and symlinking ```libjvm.dylib``` to the old location.

It worked, it fixed the launching! (**_and Minecraft works as-is for me now, no more modifying the .app bundle!!_**)

I've since [updated the original code](https://gist.github.com/pudquick/349f063c242239952a2e), so that both Rich's blog post and this one here provide an all-in-one solution.

As always, hope this helps you 😄

{% gist pudquick/349f063c242239952a2e %}

*- mike*
