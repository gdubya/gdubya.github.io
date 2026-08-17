---
layout: post
title:  "Rebuilding the PenguinPool commentary as an MLflow agent"
date:   2026-07-02 01:22:21 +0200
categories: databricks mlflow genie llm
---

A few weeks ago I shared how I moved the Bouvet PenguinPool from Erik Stubø's legendary Excel sheets to [a fully automated commentary system on Databricks][previous-post]. The jobs run, the Slack messages land, the penguin fans are happy.

But a nagging thought lingered: the automation was a notebook. A very good notebook, but still a script. No versioning, no API contract, no easy way to call it from anywhere other than a scheduled job.

So this evening I went back in — with a little help from Genie Code — and turned it into a proper MLflow agent.

![An illustration of a penguin in a Databricks shirt commentating on a World Cup match, alongside a human commentator reading a newspaper and a robot at a Bouvet-branded tablet]({{ site.url }}/images/penguinpool-agent-commentators.jpg)

## The agent layer: MLflow ChatAgent

The entire pipeline now lives inside an [`mlflow.pyfunc.ChatAgent`][chat-agent] class. Feed it a natural language message like "Preview tonight's evening message" and it returns a fully-formatted Norwegian Slack post.

It has a defined input/output contract, it's testable in isolation, and it behaves the same whether you call it from a notebook, a job, or an API endpoint.

## The context layer: Genie and Tavily

Before the LLM sees anything, the agent assembles context from two sources.

Genie queries the lakehouse for live match results, current standings, and prediction patterns from the pool participants. [Tavily][tavily] then runs targeted web searches to pull in recent news articles about the specific teams playing that evening, adding the kind of colour that makes a commentary feel current rather than purely statistical.

## The registry layer

The agent is registered in [Unity Catalog][uc-models]. Any notebook, job, or endpoint in the workspace can load it with three lines of code. Versioned, aliased, auditable.

## The observability layer: MLflow Tracing

Every run generates a [structured trace][tracing] across all spans, including the Genie queries, the web search and the LLM call.

![An MLflow trace for the agent, showing spans for querying Genie, searching web news, fetching recent messages and generating the Slack message]({{ site.url }}/images/penguinpool-mlflow-trace.jpg)

Three runs in, average end-to-end latency sits around 70 seconds, with the two sequential Genie queries as the obvious bottleneck. Now that it's properly instrumented, that's a very measurable thing to fix.

## Worth it?

Same two daily Slack messages as before. Same Norwegian sportskommentator energy. But now the thing generating them is a registered, versioned, loadable model you can call over REST.

Is it overkill for a football prediction pool? Almost certainly. Is it a satisfying evening's work? Absolutely.

*This post first appeared [on LinkedIn][linkedin-original].*

[previous-post]: {{ site.baseurl }}/posts/2026/06/13/penguinpool-commentary-analyst/
[chat-agent]: https://learn.microsoft.com/en-gb/azure/databricks/agents/custom-agents/author-agent
[tavily]: https://www.tavily.com/
[uc-models]: https://learn.microsoft.com/en-gb/azure/databricks/machine-learning/manage-model-lifecycle/
[tracing]: https://learn.microsoft.com/en-gb/azure/databricks/mlflow3/genai/tracing/
[linkedin-original]: https://www.linkedin.com/feed/update/urn:li:activity:7478226520113848320/
