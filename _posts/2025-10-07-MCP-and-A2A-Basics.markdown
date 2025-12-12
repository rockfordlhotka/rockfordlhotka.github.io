---
layout: post
title: MCP and A2A Basics
postDate: 2025-10-07T17:39:53.3750407-05:00
categories: []
tags: []
published: true
permalink: 
image: 
---
I have been spending a lot of time lately, learning about the Model Context Protocol (MCP) and Agent to Agent (A2A) protocols. And a little about a slightly older technology called the activity protocol that comes from the Microsoft bot framework.

I'm writing this blog post mostly for myself, because writing content helps me organize my thoughts and solidify my understanding of concepts. As they say with AIs, mistakes are possible, because my understanding of all this technology is still evolving.

(disclaimer: unless otherwise noted, I wrote this post myself, with my own fingers on a keyboard)

## Client-Server is Alive and Well

First off, I think it is important to recognize that the activity protocol basically sits on top of REST, and so is client-server. The MCP protocol is also client-server, sitting on top of JSON-RPC.

A2A _can be_ client-server, or peer-to-peer, depending on how you use it. The sipmlest form is client-server, with peer-to-peer provide a lot more capability, but also complexity.

## Overall Architecture

These protocols (in particular MCP and A2A) exist to enable communication between LLM "AI" agents and their environments, or other tools, or other agents.

### Activity Protocol

![Bot Framework logo]({{ site.url }}/assets/2025-10-07-MCP-and-A2A-Basics/bot-framework.png)

The activity protocol is a client-server protocol that sits on top of REST. It is primarily used for communication between a user and a bot, or between bots. The protocol defines a set of RESTful APIs for sending and receiving activities, which are JSON objects that represent a message, event, or command. The activity protocol is widely used in the Microsoft Bot Framework and is supported by many bot channels, such as Microsoft Teams, Slack, and Facebook Messenger.

(that previous paragraph was written by AI - but it is pretty good)

### MCP

![MCP logo]({{ site.url }}/assets/2025-10-07-MCP-and-A2A-Basics/mcp-logo.png)

The Model Context Protocol is really a standard and flexible way to expand the older concept of LLM tool or function calling. The primary intent is to allow an LLM AI to call tools that interact with the environment, call other apps, get data from services, or do other client-server style interactions.

The rate of change here is pretty staggering. The idea of an LLM being able to call functions or "tools" isn't that old. The limitation of that approach was that these functions had to be registered with the LLM in a way that wasn't standard across LLM tools or platforms.

MCP provides a standard for registration and interaction, allowing an MCP-enabled LLM to call in-process tools (via standard IO) or remotely (via HTTP).

If you dig a little into the MCP protocol, it is erily reminiscent of COM from the 1990's (and I suspect CORBA as well). We provide the LLM "client" with an endpoint for the MCP server. The client can ask the MCP server what it does, and also for a list of tools it provides. Much like `IUnknown` in COM.

Once the LLM client has the description of the server and all the tools, it can then decide when and if it should call those tools to solve problems.

You might create a tool that deletes a file, or creates a file, or blinks a light on a device, or returns some data, or sends a message, or creates a record in a database. Really, the sky is the limit in terms of what you can build with MCP.

### A2A

![A2A Protocol]({{ site.url }}/assets/2025-10-07-MCP-and-A2A-Basics/a2a_protocol.jpg)

Agent to Agent (A2A) communication is a newer and more flexible protocol that (I think) has the potential to do a couple things:

1. I could see it replacing MCP, because you can use A2A for client-server calls from an LLM client to an A2A "tool" or agent. This is often done over HTTP.
2. It also can be used to implement bi-directional, peer-to-peer communication between agents, enabling more complex and dynamic interactions. This is often done over WebSockets or (better yet) queuing systems like RabbitMQ.

## Metadata Rules

In any case, the LLM that is going to call a tool or send a message to another agent needs a way to understand the capabilities and requirements of that tool or agent. This is where metadata comes into play. Metadata provides essential information about the tool or agent, such as its name, description, input and output parameters, and more.

"Metadata" in this context is human language descriptions. Remember that the calling LLM is an AI model that is generally good with language. However, some of the metadata might also describe JSON schemas or other structured data formats to precisely define the inputs and outputs. But even that is usually surrounded by human-readable text that describes the purpose of the scheme or data formats.

This is where the older activity protocol falls down, because it doesn't provide metadata like MCP or A2A. The newer protocols include the ability to provide descriptions of the service/agent, and of tool methods or messages that are exchanged.

## Authentication and Identity

In all cases, these protocols aren't terribly complex. Even the A2A peer-to-peer isn't that difficult if you have an understanding of async messaging concepts and protocols.

What does seem to _always_ be complex is managing authentication and identity across these interactions.

There seem to be multiple layers at work here:

1. The client needs to authenticate to call the service - often with some sort of service identity represented by a token.
2. The service needs to authenticate the client, so that service token is important
3. HOWEVER, the service also usually needs to "impersonate" or act on behalf of a user or another identity, which can be a separate token or credential

Getting these tokens, and validating them correctly, is often the hardest part of implementing these protocols. This is especially true when you are using abstract AI/LLM hosting environments. It is hard enough in code like C#, where you can see the token handling explicitly, but in many AI hosting platforms, these details are abstracted away, making it challenging to implement robust security.

## Summary

The whole concept of an LLM AI calling tools and then service and then having peer-to-peer interactions has evolved very rapidly over the past couple of years, and it is _still_ evolving very rapidly.

Just this week, for example, Microsoft announced the [Microsoft Agent Framework](https://learn.microsoft.com/en-us/agent-framework/overview/agent-framework-overview) that replaces Semantic Kernel and Autogen. And that's just one example!

What makes me feel better though, is that at their heart, these protocols are just client-server protocols with some added layers for metadata. Or a peer-to-peer communication protocol that relies on asynchronous messaging patterns.

While these frameworks (to a greater or lesser degree) have some support for authentication and token passing, that seems to be the weakest part of the tooling, and the hardest to solve in real-life implementations.
