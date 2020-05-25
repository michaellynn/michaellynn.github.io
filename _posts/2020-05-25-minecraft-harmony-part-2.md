---
summary: "Minecraft Harmony (Bedrock + Java), Revisted"
layout: post
tags: osx macos java minecraft
date: "2020-05-25 08:00:00"
title: Minecraft Harmony Revisited
---

In [my original blog post](/2020/03/28/minecraft-harmony/) on getting Minecraft Java and Bedrock editions playing together, I provided some example scripts and configurations which inadvertently made something required for the Bedrock players that **isn't strictly required**: Namely a Minecraft paid-for Mojang account.

This was purely an accident on my part. I *thought* it would be possible to do this, but in my testing I couldn't figure out why it wasn't working, so I just published the instructions saying it was required.

## Here's where I messed up:

{% gist pudquick/af8be79d6c1308fc430cac8d69757c43 server.sh %}

See that `-o true` part?

That means: Force online mode (aka require connecting users have a valid Minecraft Java account)

I didn't realize that my server startup instructions were forcing online mode (regardless of server setting configuration).

Online accounts are an artefact of how Mojang sells Java Minecraft. The software is freely available to download, so there's no DRM in it - but in order to start playing it required that you login with a paid account. Additionally, this helped with hosted servers because you could ban/block specific accounts or make a server whitelisted only to the particular accounts of your friends.

**If you're playing at home, however** and not publicly making the server available to others - this is less of a concern!

While you'll still need accounts for your true Minecraft Java players (to launch and start Minecraft), to add in your Bedrock/mobile players you don't need additional accounts.

## How to fix it:

Edit your `server.sh` and remove the `-o true` part.

That line should now look something like:
```bash
java -Xmx1024M -jar craftbukkit-1.15.2.jar
```

Then, after running your server the first time you should have a `server.properties` file in the same directory as the server jar.

Make sure you've stopped your server, then edit the `online-mode` line in the file to say:

`online-mode=false`

## The effect of this change:

Now, when you see this dialog:
![](</images/2020-03-28-minecraft-harmony/auth.png>)

**Contrary to any other warnings about "required" you might see**, you can literally put any username you like here. My youngest daughter *really* enjoys this because now her character has her real name (whereas normally paid Minecraft account usernames are unique and may already be owned by someone else).

For the password - just leave it blank. Then hit Submit!

<small>[ **Note:** If you do this, **please don't put this Minecraft server online** by setting up port forwarding, as now anyone can connect and since they can choose any arbitrary username, there's no way to ban or whitelist which users are allowed. ]</small>

## Other side effects:

Since people logging into the server are now no longer connected to an authenticated Java Minecraft account, there's no metadata being delivered about what skin to use for the characters. As such your characters will end up with a random Steve or Alex skin.

## Additional things you can clean up while you're here:

Remember this part?

```
*** Error, this build is outdated ***
*** Please download a new build as per instructions from https://www.spigotmc.org/go/outdated-spigot ***
*** Server will start in 20 seconds ***
```

You can fix that by adding a new argument to the `server.sh` script so that it starts up instantly:

`-DIReallyKnowWhatIAmDoingISwear=true`

So now your combined line with both of these changes looks something like:

`java -Xmx1024M -DIReallyKnowWhatIAmDoingISwear=true -jar craftbukkit-1.15.2.jar`


## Changes since the last blog post:

The branch I prefered using for GeyserMC was merged back into the main branch. As such, the link I had previously no longer works.

You can now simply use the master branch, available here: [https://ci.nukkitx.com/job/GeyserMC/job/Geyser/job/master/](https://ci.nukkitx.com/job/GeyserMC/job/Geyser/job/master/)

Keep on having fun and staying sane out there.

As always, hope this helps.

*- mike*