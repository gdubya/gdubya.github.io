---
layout: post
title: "From Zero to the Garmin Store in One Evening: Vibe-Coding 'Avgang'"
date: 2026-11-04 22:00:00 +0100
categories: [side-projects, wearables]
tags: [garmin, monkey-c, antigravity, entur, norway]
redirect_from: /posts/2026/11/04/avgang-garmin-entur-widget/
---

![Avgang Garmin widget showing live Entur bus departures]({{ site.url }}/images/avgang-hero-image.png)

Many a weekday morning I find myself standing in the hallway with my coat and shoes on, wondering if I need to sprint for the bus or if I have three minutes to spare. 

I can get out my phone, unlock it, find the Ruter app, and the departure times, but when you are halfway out the door, that is just enough friction to be annoying.

Sitting on my wrist is a Garmin smart watch. It tracks my runs and sleep, so why couldn't it show me the next bus departure from the stop down the road?

## Monkey C, monkeydo

Garmin devices run on Monkey C, Garmin's proprietary object-oriented language. 

On older watches like the Fenix 6S, you have to deal with tight memory budgets (often under 64KB for widgets), asynchronous web requests, and custom canvas drawing. I had never written a single line of Monkey C in my life.

Normally, the prospect of installing a new SDK, reading documentation, and debugging memory leaks for a micro-app means the idea stays firmly on the backlog. 

I decided to see what would happen if I vibe-coded it with Google Antigravity instead.

## Building "Avgang" in 45 Minutes

I started with a clean slate: no boilerplate, just the idea, the watch, and the [Entur open API][entur-api].

Antigravity helped piece together the entire project:
* Generating the Monkey C project structure and manifest files.
* Writing the `Communications.makeWebRequest` callback to fetch departures asynchronously.
* Trimming down the Entur payload so the JSON parser did not choke on the Fenix 6S memory limit.
* Laying out the UI canvas for both the glance view (the quick-scroll widget list) and the full 240x240 screen.

Within 45 minutes, I had a working prototype running in the simulator and pulling live departure times. The rest of the evening went into the finishing touches, and by bedtime Avgang (Norwegian for "departure") was submitted and published to the Garmin Connect IQ Store.

👉 **[Avgang on the Garmin Connect IQ Store][avgang-store]**

## Why This Matters

Vibe-coding with Antigravity enabled me to turn an idea into a real app in practically no time, without needing to orchestrate an entire army of agents or burn millions of tokens. 

Monkey C is a niche language, and I probably would never have spent a weekend wading through SDK documentation just to build a tiny convenience widget for myself. Going from zero experience to a published store app in one evening is hard to argue with.

If you live in Norway and wear a Garmin, give [Avgang][avgang-store] a try.

[entur-api]: https://developer.entur.org/
[avgang-store]: https://apps.garmin.com/apps/1da1d892-3d10-4184-b0d4-2be9df04b91b
