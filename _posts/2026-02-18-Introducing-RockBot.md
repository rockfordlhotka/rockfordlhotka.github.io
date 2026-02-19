---
layout: post
title: Introducing RockBot
postDate: 2026-02-18T00:00:00-06:00
categories: []
tags: [ai, agents, rockbot, dotnet]
published: true
permalink: 
image: /assets/2026-02-18-Introducing-RockBot/rockbot-image.png
---

![RockBot](/assets/2026-02-18-Introducing-RockBot/rockbot-image.png)
I've been working on a new project called [RockBot](https://github.com/MarimerLLC/rockbot), a framework for building agent and multi-agent AI systems where agents and user proxies communicate exclusively through a message bus in a cloud-native architecture.

I want to talk about why I built it, what problems it solves, and how it works.

## Not Written Here Syndrome

[OpenClaw](https://openclaw.ai/) kind of took the world by storm when it launched, and is really amazing. I found it very inspirational, but all the (reputed) security issues have made me hesitant to really run it, much less dig in deep.

[nanobot](https://github.com/HKUDS/nanobot) was inspired by OpenClaw, and seems to be more focused on security and isolation. I actually set up and have been running a nanobot instance for a while, but it has been flaky, at least for me. Lots of hung conversations.

With nanobot being under 4000 lines of code, I thought to myself "how hard can it be to build something like this myself?" I have a lot of experience building distributed systems, and I thought I could apply that knowledge to build a more robust and secure agent framework.

## The Problem with Most AI Agent Frameworks

Most AI agent frameworks today run LLM-generated code in the same process as the host application. That sounds convenient, but it creates serious problems:

- LLM-generated code can access the host directly — file system, network, secrets, everything in the process
- Swapping LLM providers or tool backends requires invasive changes throughout the codebase
- One runaway agent can crash or compromise the entire system
- Scaling individual components is impractical when everything runs together

When I started thinking about building autonomous agents, these problems felt fundamental. Not quirks to work around, but architectural failures that need to be solved at the design level.

Maybe it is because I've built my career around enterprise software, specifically focused on distributed systems. I want something that, while not perfect, at least follows the architectural principles that have proven to work in large-scale, production-grade software. That means separation of concerns, isolation of execution, least privilege, and cloud-native design.

## The Message Bus Architecture

RockBot is built around one central idea: agents communicate exclusively through a message bus. There is no shared memory, no direct method calls between agents, and no LLM-generated code running in-process with the host.

Each agent is an isolated process that:

1. Subscribes to topics on the message bus
2. Receives messages
3. Invokes tools or calls LLMs as needed
4. Emits responses back onto the bus

The message bus is backed by RabbitMQ in production, though that can be swapped out for other implementations if needed. It is easy enough to run a RabbitMQ instance in Docker Desktop for local development.

This isn't a new idea. Event-driven architecture has been around for decades. What's new is applying it rigorously to AI agent systems, where the isolation properties matter even more than in traditional software.

## Four Design Goals

RockBot is built around four foundational goals:

**Separation of concerns.** Every responsibility has a clear owner with a well-defined boundary. Agents handle reasoning. The message bus handles routing. Tool bridges handle execution. LLM providers handle inference. None of these cross into each other's domain — they communicate only through typed messages. This makes each layer independently testable, replaceable, and understandable.

**Isolation of execution.** LLM-generated code never runs in the same process as the host. Agents run in separate processes with no shared memory. Scripts execute in ephemeral Kubernetes containers that are discarded after use. A compromised or runaway agent cannot access the host, read its secrets, or affect other agents. Failure is contained by design.

**Principle of least privilege.** Each component knows only what it needs to do its job. Agents receive only the messages addressed to them. Tool bridges expose only the tools they are explicitly configured to serve. Scripts run in containers with no network access, no persistent storage, and no credentials. No component accumulates capabilities beyond its immediate task.

**Cloud-native by design.** Outside the core agent itself, workers are stateless — state lives in messages or external stores, never in process memory. Components scale independently. The message bus provides back-pressure, dead-letter queues, and durability so the system degrades gracefully under load. Configuration flows from the environment, secrets from Kubernetes Secrets, and observability out through OpenTelemetry.

## Core Components

The framework is composed of several focused packages. Here are some key ones:

| Package | Purpose |
| --- | --- |
| RockBot.Messaging.Abstractions | Transport-agnostic contracts (IMessagePublisher, IMessageSubscriber, MessageEnvelope) |
| RockBot.Messaging.RabbitMQ | RabbitMQ provider with topic exchanges and dead-letter queues |
| RockBot.Messaging.InProcess | In-memory bus for local development and testing |
| RockBot.Host | Agent host runtime — receives messages and dispatches through the handler pipeline |
| RockBot.Llm | LLM integration via Microsoft.Extensions.AI |
| RockBot.Tools / RockBot.Tools.Mcp | Tool invocation — REST and MCP (Model Context Protocol) |
| RockBot.Scripts.Container | Ephemeral script execution in isolated Kubernetes containers |
| RockBot.A2A | Agent-to-agent task delegation over the message bus |
| RockBot.Cli | Unified host application — runs agents as hosted services |

The abstractions layer is deliberately thin. If you want to swap RabbitMQ for a different message broker, you implement a new provider against the same contracts and nothing else changes.

## Agent Profiles

One thing I particularly like about this design is how agents are configured. Each agent gets three markdown files:

- `soul.md` — Core identity, values, and goals
- `directives.md` — Behavioral rules and constraints  
- `style.md` — Tone, formatting, and communication style

These files are human-readable, version-controlled, and composed at runtime into the agent's system prompt. This means you can adjust an agent's behavior without touching code, just by editing its profile files. It also means your agent configuration is right there in source control alongside your code, reviewable and auditable by the whole team.

## Bridges to the World

RockBot supports [Model Context Protocol (MCP)](https://modelcontextprotocol.io/) for tool integration. Place an `mcp.json` file alongside `RockBot.Cli` and the MCP bridge will discover and register available tools automatically. This means any MCP-compatible tool server can be plugged in without writing glue code.

This is particularly useful because the MCP ecosystem is growing rapidly. If a tool has an MCP server, RockBot can use it.

The MCP bridge shields the agent from having to know details about all the MCP servers that may be registered. It maintains and provides a list of available MCP servers, and only provides tool details on-demand. This avoids overloading the agent's context window with information about tools it may never use, while still making them available when needed.

RockBot also supports web searching and browsing, which allows the agent to search and read web content. Typically this content gets pulled back into searchable working memory so the agent can reason over it without needing to access the web repeatedly.

And there is support for executing arbitrary Python scripts in isolated containers. This allows agents to perform complex computations, data processing, or interactions with external systems without risking the host's security.

## Memory and State

The agent itself does have persistent memory at several levels:

1. Short-term working memory is passed in the system prompt and updated with each message. This is where the agent's current context lives.
2. Long-term memory is stored as a set of files in a folder structure defined by the agent. These files include tags, and support lexical retrieval to pull relevant information into the agent as needed.
3. Working memory is in-memory and ephemeral. This is where the agent can stash blobs of information from the web, MCP servers, or after running scripts. Think of this as a short-term cache that the agent can use to hold information it is actively reasoning over, without needing to write it to disk or include it in the system prompt.

## Skills

The agent can maintain its own set of skills: markdown files with instructions on how to do whatever the agent needs to do. I'm finding that it is often very beneficial to have a skill for each MCP server or each workflow the agent needs to perform. This allows the agent to learn how to use tools over time, and to have a clear place to look up instructions on how to do things.

## User Interaction

There are two user proxies at the moment: a CLI tool for testing, and a Blazor web UI for actual use.

The user proxy concept is extensible, so it is quite possible to create all sorts of other user interfaces.

A user proxy communicates with the agent via RabbitMQ messages, just like any other agent or component. This means the user interface is completely decoupled from the agent's reasoning and tool execution. The UI can be swapped out, scaled independently, and even run on a different machine or in a different environment without affecting the core agent logic.

## What's Next

The project is still early. The core architecture is in place, the RabbitMQ and in-process transports work, LLM integration is done via Microsoft.Extensions.AI, and the Blazor Server UI is functional. There's plenty more to build, and the design is intentionally extensible.

If you're interested in multi-agent AI systems and care about security, isolation, and cloud-native deployment, I'd love for you to take a look. The repository is at [https://github.com/MarimerLLC/rockbot](https://github.com/MarimerLLC/rockbot). Contributions are welcome — just open an issue before starting significant work so we can discuss the approach.

The AI agent space is moving fast. My goal with RockBot is to build something that makes it possible to create serious, production-grade agentic systems without sacrificing the security and architectural principles that make software sustainable over time.
