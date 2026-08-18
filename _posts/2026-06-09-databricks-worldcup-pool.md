---
layout: post
title:  "Building a World Cup pool with Databricks Apps, Lakebase and Genie"
date:   2026-06-09 19:56:00 +0200
categories: [databricks]
tags: [apps, lakebase, genie]
redirect_from: /posts/2026/06/09/databricks-worldcup-pool/
---

Recently Douglas Garcia Torres shared an open-source project created by Onno van der Horst for running a [World Cup pool as a Databricks App][worldcup-pool]. Naturally, this gave me the perfect excuse to spend some time playing around with Databricks Apps, Lakebase, Genie, and a few of the newer platform capabilities. A couple of evenings later we have an internal football pool up and running at Bouvet.

Here's what I learned along the way.

## Databricks Apps are easy to get running

Surprisingly easy, in fact. I made a few small tweaks to get the sync jobs running properly in a [Serverless Workspace][serverless-ga], but overall the experience was very smooth.

## Lakebase CDF has a few limitations, for now

The brand-new [Lakebase Change Data Feed][lakebase-cdf] functionality doesn't currently support default storage, which is what Serverless Workspaces use. It's still in Public Preview, so I'd expect this to move — but as things stand the destination catalog can't be one backed by default storage.

Fortunately the project already included jobs that sync the Postgres database into Delta tables, so it was easy enough to work around.

## Customising the app is refreshingly simple

I added a "Guess all results" button for the less... detail-oriented participants. Because honestly, manually entering and tweaking predictions for all 104 matches can consume an entire tea break. Or two.

## Genie adds a really interesting layer

Adding a [Genie][genie] Space on top of the pool data created a surprisingly fun conversational analytics experience:

> Who has the most accurate predictions?
>
> Which matches caused the biggest ranking changes?
>
> Who still has a realistic chance of winning?

It's a good example of how natural language interfaces can make even simple datasets much more engaging.

## Scheduled tasks for Genie are very handy

The new scheduled task functionality for Genie Spaces made it easy to set up daily updates of the standings and key insights automatically.

## Worth the couple of evenings

What started as a small side project turned into a really fun way to explore some of the newer Databricks capabilities in a practical scenario. Now I'm mostly looking forward to the football, the data, the analytics, and the friendly competition — and probably overanalysing everybody's predictions.

Big thanks again to Douglas Garcia Torres and Onno van der Horst for the inspiration and the [open-source project][worldcup-pool].

*This post first appeared [on LinkedIn][linkedin-original].*

[worldcup-pool]: https://github.com/onno101/worldcup-pool
[serverless-ga]: https://www.databricks.com/blog/serverless-workspaces-azure-databricks-now-generally-available
[lakebase-cdf]: https://learn.microsoft.com/en-gb/azure/databricks/oltp/projects/lakebase-cdf
[genie]: https://learn.microsoft.com/en-gb/azure/databricks/genie-one/chat
[linkedin-original]: https://www.linkedin.com/feed/update/urn:li:activity:7470171863327850496/
