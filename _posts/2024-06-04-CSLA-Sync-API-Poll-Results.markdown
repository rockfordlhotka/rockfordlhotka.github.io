---
layout: post
title: CSLA Sync API Poll Results
date: 2024-06-04T00:00:00.0000000-05:00
categories: []
tags: []
published: true
permalink: 
image: 
---
A few weeks ago I posted about the idea of [removing the sync APIs from a future version of CSLA .NET](https://blog.lhotka.net/2024/05/23/Deprecating-CSLA-Synchronous-APIs). That post included a poll, and this post will summarize the results of that poll.

I really appreciate the people who took the time to answer the poll, thank you!

**This post isn't the decision point about the sync API question - just a summary of the poll results.**

## What UI Frameworks Do You Current Target

![What UI Frameworks Do You Current Target]({{ site.url }}/assets/2024-06-04-CSLA-Sync-API-Poll-Results/q1.png)

These results are interesting in my view, though not entirely surprising.

CSLA provides the most value in a scenario where the client and server are both written in .NET, which includes Blazor, MAUI, WPF, and Windows Forms.

I'm pleased to see [Uno](https://platform.uno) in there too, as I really like their tooling and capabilities.

It is also no surprise that people use web server technologies in the ASP.NET and ASP.NET Core families, as those were very popular in that window of time after the iPad and before SPAs became viable.

## What Versions of CSLA Are You Using

![What Versions of CSLA Are You Using]({{ site.url }}/assets/2024-06-04-CSLA-Sync-API-Poll-Results/q2.png)

I'm very pleased to see the number of people using CSLA 8, though I suspect that is skewed from the world-at-large because people who are using the latest versions are probably also more engaged in the CSLA community.

It also makes sense that a lot of folks are on CSLA 5, because the hard requirement to use dependency injection starting at CSLA 6 is a very real barrier to upgrading.

## Do You Currently Use Synchronous API Calls

![Do You Currently Use Synchronous API Calls]({{ site.url }}/assets/2024-06-04-CSLA-Sync-API-Poll-Results/q3.png)

Not surprising, only a small number of people exclusively use async APIs. Probably in part because CSLA still has a lot of sync APIs to use.

I _suspect_ (based on the comments field) that most people who _exclusively_ use sync APIs are maintaining WinForms/WPF apps, where moving to async would require a lot of rework of the overall app flow and user experience.

## Is Modernizing Your Codebase to Async APIs Realistic

![Is Modernizing Your Codebase to Async APIs Realistic]({{ site.url }}/assets/2024-06-04-CSLA-Sync-API-Poll-Results/q4.png)

![Question 4 insights]({{ site.url }}/assets/2024-06-04-CSLA-Sync-API-Poll-Results/q4-insights.png)

17% indicate that it would be cost prohibitive to move entirely away from sync APIs.

An interesting anecdote here is a comment I got from Immo on the .NET product team via twitter:

![Twitter post from Immo]({{ site.url }}/assets/2024-06-04-CSLA-Sync-API-Poll-Results/immo-twitter.png)

Rather than removing sync APIs, the .NET team is adding _more_ sync APIs alongside async APIs.

## What Is a Realistic Timeframe for CSLA to Require Async APIs

![What Is a Realistic Timeframe for CSLA to Require Async APIs]({{ site.url }}/assets/2024-06-04-CSLA-Sync-API-Poll-Results/q5.png)

Over half the responses indicate sync APIs could be removed within a couple years.

About a third of folks think a 3+ time frame is more realistic.

12% say we should never drop sync APIs.

## If a future version of CSLA .NET only supported async APIs would you continue to use CSLA

![If a future version of CSLA .NET only supported async APIs would you continue to use CSLA]({{ site.url }}/assets/2024-06-04-CSLA-Sync-API-Poll-Results/q6.png)

Not surprisingly 12% say they'd have to stop using CSLA if it had no sync APIs. I don't know if this means they'd stay on the previous version "forever", or if this would trigger a migration totally away from CSLA.

Interestingly, ~13% use Windows Forms, and when I've suggested dropping support for that (off and on) over the years, there's a lot of push back that has led me to maintain support.

Also interesting is that maintaining Windows Forms support is _way harder_ than maintaining the sync APIs, because there's a lot of nasty crusty code to make WinForms data binding work.

## Conclusion

This post isn't the decision point on whether to keep the sync APIs in CSLA. It is just my opportunity to publish the results of the survey so everyone can enjoy.