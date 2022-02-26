---
summary: "Necessity is the mother of invention"
layout: post
tags: agony networking
date: "2022-02-25 08:00:00"
title: The Internet is a Series of Cables
---

I just had to write this one up, because everyone deserves to know stupid tricks that work.

This is also a story of my reason for needing to do something this stupid. If you just want to get to the stupid trick part, you can [skip ahead](#stupidtrick). If you want to feel a little of my pain, then read on.

---

If you don't follow me on [twitter](https://twitter.com/mikeymikey), you probably didn't see that last fall we were fortunate enough to move into a newly built home. And it's just lovely, really it is.

And like anything worth doing well, it took time.

When we started well over a year ago in our goal of finding/making our new home, I had three basic needs:
 - The mortgage needs to be reasonable
 - I need a dedicated office in the house
 - The house needs to support high speed internet (by which I mean cable or fiber, not DSL)

That last bulletpoint was actually critical in passing on a few locations we were looking at. Just about every single address will always pass for DSL (as long as you're not out in the total middle of nowhere), but having worked for a DSL reseller in the past I was all too familiar with the wide variations in speed and how absolutely crucial it was to be *very* close to the nearest central office for anything remotely speedy (Dearest reader: We were not planning to live close to *any* CO).

As it turns out, checking for high speed internet service where a house doesn't actually exist yet is somewhat non-trivial. **I thought** it was good enough to check whether service was available at every other address on the street.

... This was, in fact, not enough.

We had already almost completed build, were waiting on some last few things, and were rapidly approaching obtaining the last inspections for occupancy when I decided "Looks like the perfect time to start getting that cable internet connected!"

I knew immediately I was headed for problems when my freshly minted (~ 1-2 months) address didn't register as an address for even a basic service check.

So I called them up.

**Them:** "Oh! It looks like your address isn't in the system yet. We'll need to get you scheduled for a service availability check for your site location."

**Me:** "Sounds great, let's do that!" So I provide all my contact information - surely I will learn that this was just a blip on my journey to the information super highway.

Time passes. 1 week. 2 weeks. Nothing.

So I call back to check on how that's going.

**Them:** "Oh! It looks like the check was done. You should have been told that service might be available at your location, but they'd have to do an assessment to see what work would be required."

**Me:** "Oh. Ok. Let's do that." I am informed this may take several more weeks. I express my concern that I didn't hear anything back last time and I'm doubly assured that this time I will surely get a call back.

... History repeats itself. I call back, again irritated at having heard nothing.

**Them:** "Oh! Hmm. It looks like this was done. You didn't hear anything? Let me see if I can get the results."

**Them:** "Ah. It says here that they will have to run a new buried service line to your address."

You see, as it turns out, even though the entire neighborhood had service, this cable company had decided that it was a brilliant idea to not actually provide cabling to all lots in the area when service was initially introduced but to instead only do it for locations which at the time had existing houses.

**Me:** "Oh, that should be fine. We had conduit put in to the house from the propery edge, so just run it through that."

**Them:** "Well, it looks like the place where service is located at is actually half way down the street from you. They would need to run a cable from that location to your conduit. **It looks like they estimate it could cost as much as $14,000**."

I now understood why they hadn't called me back.

They had a shut-the-fuck-up-and-go-away back of the napkin estimate and they expected I'd balk when hearing it. Either that or they were still just very bad at customer service and had once again failed to get back to me.

But what the cable company didn't know was that the house construction was under budget.

We had the money to do this.

**Me:** "Ok! Let's do it!"

... There was momentary silence on their end. I don't think they get this response very often. I think they were a bit lost on what happens next.

**Them:** "Ohhh. Uh. Ok. We'll need your contact information to get an initial survey scheduled with actual pricing for the estimated work involved."

So I gave them my contact information. For the third time. I expected to never hear from them again. I was mostly right.

I heard from someone, but it wasn't the cable company. It was a third-party escrow company who was contracted to hold onto the funds on my behalf for the initial survey and the actual work to be done.

This person reached out to me themselves. They were informative. They were easy to get in contact with. They were responsive! All of the things the cable company had not been so far. It was a pleasure talking to them.

Working with them, an initial assessment was done by someone who showed up and took photos of the street and portions of my property. After they drew up the plans of what would need to be done and those were sent back, a grand total closer to about $11,000 was finally settled on. I went through a series of electronic documents, paid the money to escrow, and then got the saddest news.

**Nice person:** "I just want to let you know, this could take up to 6 months."

6 months ?!?!!? To go like 200 feet to my conduit !?!?

Meanwhile, we were already in the process of selling the old house as we'd just passed final occupancy inspection for the new one.

---

<p id="stupidtrick">So I'm in a bind. I need internet - for work, for the kids, for family harmony.</p>

But it could be 6 months before it gets here? What do I do??

Starlink was out. I live in a forest, no good view on the sky.

Cellular? I get 1 bar of LTE on a **good** day.

**Idea:** Maybe if I covered a neighbor's bill, I could use theirs? Turns out - yes! One of them was willing!

Now - we wanted to live where we weren't directly in our neighbor's business. We have a couple acres. We've got several hundred feet between ourselves and the remote beginnings of anyone next door.

... so maybe run a cable?

**Problem:** The ethernet standard, if you're not familiar with it, promises you 100 meters before you'll need a repeater. In feet, that's just shy of 330 feet. And my neighbor was easily 2x that. A repeater isn't going to cut it. I'd need power half way down the line - maybe twice. And outdoor rated repeaters. More connections, more problems - sounds like a recipe for disaster.

How about coaxial? Have you heard of [MoCA](https://en.wikipedia.org/wiki/Multimedia_over_Coax_Alliance)? It sounds like it would be perfect. But no, it generally has the same distance limitations of ethernet (at least for the consumer gear you're likely to get your hands on).

And fiber wasn't the best of ideas either. Besides being expensive and hard to work with, I had concerns about it surviving outdoors for months on end above ground in a forest (I sure as heck wasn't going to bury it the whole way).

... so what about wireless point to point?

**New problem:** Like I said, I live in a forest:

![](</images/2022-02-25-the-internet-is-a-series-of-cables/my_view.jpg>)

Line of sight wasn't really doable.

So what to do?

.. as it turns out, DSL *was* actually the solution here. But not in the way you'd expect.

You can, in fact, become your own DSL provider(ish) with a [pair of DSL transceivers](https://www.amazon.com/Tupavco-Ethernet-Extender-Kit-Repeater-VDSL/dp/B01BOD8C9W/) like these!

<center><img src="/images/2022-02-25-the-internet-is-a-series-of-cables/magicbox.jpg"/></center>

They effectively act like a patch cable, passing traffic from one end to the other at up to 100 Mbps and up to 7000 feet (though not at that speed at full length). All you need is Cat5 or better inbetween and power at both ends. Ordered a 1000 ft spool of it and we're good to go. Perfect!

So I can wire up one end of this in my house, connected to my home router and APs - but what about the other end?

... How do I get something easily connected to a neighbor's internet but without inconveniencing them when they've already been so generous?

The answer was in using a piece of equipment that I'd long been a fan of, pre-pandemic, when traveling - a [GL.iNet travel router](https://www.gl-inet.com/products/gl-mt1300/):

<center><img src="/images/2022-02-25-the-internet-is-a-series-of-cables/router.jpg"/></center>

These things are my swiss army knife when it comes to solving networking challenges. One of the modes you can put it into is as an ethernet extender for an existing SSID. When power cycled, it'll come back up and try to re-associate.

So all I needed was to connect one of these, configured to join my neighbor's guest wifi network (to keep his household traffic separate from mine) to one of the transceivers with a little power - and all I'll need to do is get it close enough to his house for a strong enough signal. No wiring required on his end aside from the power connection.

So I grabbed a [fantastic waterproof outdoor box](https://www.amazon.com/SOCKiTBOX-Weatherproof-Connection-Electrical-Transformers/dp/B00274SLK8/) (seriously, I need to buy a few more of these), combined it all together - and this wonderful little thing is what provided my household internet connectivity from October until February - 5 months! - with only a single maintenance instance required the entire time (had to power cycle it after a neighborhood power outage brought up the network in the wrong order):

<center><img src="/images/2022-02-25-the-internet-is-a-series-of-cables/internals.jpg"/></center>

<center><img src="/images/2022-02-25-the-internet-is-a-series-of-cables/installed.jpg"/></center>

(The white cable is power running into the box, the black cable is my outdoor rated Cat5 - those are the only connections made!)

The connection wasn't amazing - but it wasn't bad either! (60Mbit down, 12 up)

---

The story does have a happy ending.

I now have a very happy internet connection:

![](</images/2022-02-25-the-internet-is-a-series-of-cables/zoomzoom.jpg>)

I hope this gives someone out there some new mental tools and ideas to try against this unforgiving world - The Packets Must Flow!

*- mike*