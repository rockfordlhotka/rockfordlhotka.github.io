---
layout: post
title: CSLA 2-tier Data Portal Behavior History
date: 2025-04-21T00:00:00.0000000-05:00
categories: []
tags: []
published: true
permalink: 
image: 
---
The CSLA data portal originally treated 2- and 3-tier differently, primarily for performance reasons.

Back in the early 2000's, the data portal did not serialize the business object graph in 2-tier scenarios. That behavior still exists and can be enabled via configuration, but is not the default for the reasons discussed in this post.

Passing the object graph by reference (instead of serializing it) does provide much better performance, but at the cost of being behaviorally/semantically different from 3-tier. In a 3-tier (or generally n-tier) deployment, there is at least one network hop between the client and any server, and the object graph _must be serialized_ to cross that network boundary.

When different 2-tier and 3-tier behaviors existed, a lot of people did their dev work in 2-tier and then tried to switch to 3-tier. Usually they'd discover all sorts of issues in their code, because they were counting on the logical client and server using the same reference to the object graph.

A variety of issues are solved by serializing the graph even in 2-tier scenarios, including:

1. Consistency with 3-tier deployment (enabling location transparency in code)
1. Preventing data binding from reacting to changes to the object graph on the logical server (nasty performance and other issues would occur)
1. Ensuring that a failure on the logical server (especially part-way through the graph) leaves the graph on the logical client in a stable/known state

There are other issues as well - and ultimately those issues drove the decision (I want to say around 2006 or 2007?) to default to serializing the object graph even in 2-tier scenarios.

There is a performance cost to that serialization, but having _all_ n-tier scenarios enjoy the same semamantic behaviors has eliminated so many issues and support questions on the forums that I regret nothing.