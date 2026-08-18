---
layout: post
title:  "Analysing three years of Norwegian grocery receipts with Databricks"
date:   2026-07-17 07:59:57 +0200
categories: [databricks]
tags: [spark, gdpr, genie]
redirect_from: /posts/2026/07/17/norwegian-grocery-receipts/
---

Now that the World Cup is over (it did not come home), here's one last side project before I unplug and vanish for a few weeks.

If you live in Norway, you'll know that the grocery market is essentially a three-horse race between Norgesgruppen (Meny, Joker, and so on), REMA 1000, and Coop. Naturally, each has its own loyalty scheme and app, diligently hoarding your shopping data for their own "nefarious purposes".

The Norgesgruppen scheme, "Trumf" (what a terrible name), recently rolled out a feature providing insights into your shopping — a breakdown by category, total weight, and so on. Nice! But I wanted a single view of all my grocery spend, not just the overpriced store down the road.

The big chains aren't exactly queuing up to hand over your receipt data via a nice, clean API. But thanks to the beauty of GDPR, they don't have a choice: we have a legal right to demand our stored data via an *innsynsrapport*.

So, I did what any self-respecting data person does. I requested the data and built a Spark notebook.

![A Databricks notebook showing year-over-year comparison and cross-chain product matching by EAN barcode and name normalisation]({{ site.url }}/images/kvittering-krunchr-notebook.jpg)

Anyone can follow along in a Databricks workspace to grab their own data, run the analysis, and spin up a dashboard — the [code is on GitHub][repo]. It works perfectly fine on the [Databricks Free Edition][free-edition], so it's a great little sandbox for testing and learning.

## What did the data actually tell me?

Well, nothing earth-shattering just yet. Unsurprisingly, I use my nearest store the most, because convenience is king. The massive shops tend to happen further afield at the cheaper chains. And I spend quite a lot on clementines.

![A Databricks dashboard titled "KvitteringKrunchr: Grocery Spending Analysis", with spending figures blurred out]({{ site.url }}/images/kvittering-krunchr-dashboard.jpg)

The notebook uses Databricks' [`ai_classify()`][ai-classify] to bucket the items, and while it's a solid start, I think I'll spend some time later refining the categories to make sure the AI isn't hallucinating my grocery habits.

I've also tested out the new Databricks Genie mobile app to keep an eye on the dashboard on the go, which works brilliantly.

![The same dashboard in the Genie mobile app, showing 482 shopping trips and a spending-by-chain breakdown]({{ site.url }}/images/kvittering-krunchr-mobile.jpg)

## What's next

Right now, the dataset is a manual export and import. The goal for autumn is to build a reliable, continuous receipt ingestion pipeline. Once that's automated, the next logical step is to set up a proper Genie Agent — the artist formerly known as Genie Space — so I can query the up-to-date grocery data natively inside the mobile app more regularly.

There are also some caveats. The data is obviously a bit incomplete. It misses the random trips where we forgot to scan a loyalty card, the odd visit to alternative shops, and of course the occasional *harrytur* across the border to Sweden.

But it's a start. Give it a spin if you want to see where your money is actually going, and let me know what your data looks like.

Have a great summer!

*This post first appeared [on LinkedIn][linkedin-original].*

[repo]: https://github.com/gdubya/kvittering-krunchr
[free-edition]: https://www.databricks.com/learn/free-edition
[ai-classify]: https://learn.microsoft.com/en-gb/azure/databricks/sql/language-manual/functions/ai_classify
[linkedin-original]: https://www.linkedin.com/feed/update/urn:li:activity:7483762397355253761/
