---
layout: post
title: The RockBot Band
postDate: 2026-03-03T00:00:00-06:00
categories: []
tags: [ai, agents, rockbot, mcp, dotnet]
published: true
permalink:
image: /assets/2026-03-03-The-RockBot-Band/rockbot-band.png
---

![RockBot Band](/assets/2026-03-03-The-RockBot-Band/rockbot-band.png)

Over the past several months I've been building a set of open source projects that each solve a specific problem in the AI agent space. Individually they're useful. Together they form the foundation for building truly agentic systems that run in production environments like Kubernetes or Azure.

I want to step back and talk about how these projects fit together, because the big picture matters more than any single component.

## The Projects

Here's the lineup:

- [RockBot](https://github.com/MarimerLLC/rockbot) — A framework for building agents and multi-agent systems, designed to be cloud-native and manageable. Think of it as the runtime and architecture for agents that communicate through a message bus with full isolation and separation of concerns.
- [mcp-aggregator](https://github.com/MarimerLLC/mcp-aggregator) — A gateway that sits between your agents (or any LLM client) and all your MCP servers. Agents interact with MCP servers without consuming massive amounts of context memory, and without needing credentials or connection details for each server.
- [calendar-mcp](https://github.com/MarimerLLC/calendar-mcp) — An MCP server that provides access to multiple M365, outlook.com, and Gmail email and calendar accounts. Not just one calendar — _all_ of them. Your work calendar, client calendars, personal and family calendars. A real picture of your actual life.
- [agentregistry](https://github.com/MarimerLLC/agentregistry) — An agent registry for dynamic discovery of A2A and ACP agents, as well as MCP servers. Supports both persistent and ephemeral instances (think KEDA-scaled containers), and agent-to-agent communication via both HTTP and queued messaging.
- [researchagent](https://github.com/MarimerLLC/rockbot/tree/main/src/RockBot.ResearchAgent) — An agent built on the RockBot framework, designed to perform research tasks where results flow back to the calling RockBot agent.

Each one addresses a gap I kept running into while building agentic systems. Let me explain how they connect.

## The Agent Runtime: RockBot

Everything starts with [RockBot](https://github.com/MarimerLLC/rockbot). It provides the fundamental architecture: agents as isolated processes communicating through a message bus (RabbitMQ in production). No shared memory, no LLM-generated code running in your host process, no ability for a compromised agent to reach into another agent's state.

I wrote about RockBot's design in detail in [Introducing RockBot](/2026/02/18/Introducing-RockBot), but the key point here is that RockBot is designed from the ground up to run in containers. The framework supports both stateful and stateless agents — state can live in the agent process, in messages, or in external stores, depending on the agent's needs. Scaling is horizontal for stateless workers, and the registry understands the difference. This matters when you start composing agents together — you need the runtime to support cloud-native deployment, not fight against it.

## A Personal Agent: RockBot Agent

The RockBot framework is the foundation, but the [RockBot Agent](https://github.com/MarimerLLC/rockbot/tree/main/src/RockBot.Agent) itself is a concrete example of what you can build with it. It's a personal and professional agent designed to help you manage your life — scheduling, research, information retrieval, task coordination, and more.

Unlike the stateless worker agents that spin up, do a job, and disappear, the RockBot agent is stateful by design. It maintains persistent memory about you: your preferences, your projects, your contacts, your communication style. It remembers what you talked about last week and builds on it. This is essential for a personal agent — you don't want to re-introduce yourself every conversation.

The agent uses markdown-based profile files (soul, directives, and style) to define its identity and behavior, and it has a multi-layered memory system with short-term, long-term, and working memory. When it needs to do something beyond its own capabilities — research a topic, check your calendar, interact with external systems — it delegates to other agents and tools through the message bus and mcp-aggregator. The RockBot agent is the hub that ties everything else together from the user's perspective.

## Tool Access Without the Bloat: mcp-aggregator

Agents need tools. MCP (Model Context Protocol) is the emerging standard for giving AI systems access to external capabilities. But if you naively give an agent direct access to a dozen MCP servers, two things happen: your agent's context window fills up with tool descriptions it may never use, and every agent needs credentials for every server.

[mcp-aggregator](https://github.com/MarimerLLC/mcp-aggregator) solves both problems. It acts as a single gateway that your agent connects to. The aggregator knows about all available MCP servers, provides concise summaries, and only loads full tool details on demand. Credentials live in the aggregator, not in every agent.

I covered this in [MCP Aggregator](/2026/02/15/MCP-Aggregator), but the piece I want to emphasize here is how this fits the cloud-native story. In a containerized environment, you configure the aggregator once and every agent in the cluster can use it. Add a new MCP server? Register it with the aggregator and every agent can discover it immediately. No redeployment, no configuration changes across dozens of agent instances.

## A Real View of Your Schedule: calendar-mcp

Most calendar integrations give you access to _one_ calendar. That's fine if you only have one, but most professionals juggle multiple accounts — work (M365), personal (outlook.com or Gmail), maybe a shared family calendar, client calendars, and so on.

[calendar-mcp](https://github.com/MarimerLLC/calendar-mcp) is an MCP server that aggregates across all of these. When your agent needs to schedule something or check your availability, it sees the complete picture. Not just your work calendar, but the dentist appointment on your personal calendar and the school event on the family calendar.

This is the kind of tool that becomes much more powerful when accessed through the aggregator. The agent doesn't need to know about OAuth tokens for three different email providers. It asks the aggregator for calendar tools, the aggregator routes to calendar-mcp, and calendar-mcp handles the multi-account complexity.

## Finding Agents and Servers: agentregistry

In a static system, you can hardcode which agents exist and where to find them. In a dynamic cloud environment, that falls apart fast. Containers spin up and down. KEDA scales agents to zero when idle and back up when there's work. New agents get deployed. Old ones get retired.

[agentregistry](https://github.com/MarimerLLC/agentregistry) provides dynamic discovery for the entire ecosystem. It knows about:

- **A2A agents** — agents that communicate via Google's Agent-to-Agent protocol
- **ACP agents** — agents using the Agent Communication Protocol
- **MCP servers** — tool servers available in the environment
- **Persistent instances** — always-running services, including stateful agents like the RockBot agent that maintain long-term memory
- **Ephemeral instances** — stateless containers that scale to zero and spin up on demand (via KEDA or similar)
- **Multiple transports** — HTTP for synchronous communication, queued messaging (like RabbitMQ) for asynchronous agent-to-agent work

This is the glue that makes a multi-agent system dynamic rather than static. When RockBot needs to delegate a research task, it doesn't need a hardcoded address. It queries the registry, finds an available research agent, and sends the task — whether that agent is already running or needs to be spun up.

## Specialized Agents: researchagent

The researchagent is a concrete example of how this all comes together. Built on the RockBot framework, it's a specialized agent designed to perform research — web searches, document analysis, information synthesis — and return structured results to the calling agent.

In practice, a user asks the RockBot agent something that requires research. RockBot recognizes the need, queries the agentregistry to find a research agent, delegates the task via the message bus, and gets results back. The research agent uses mcp-aggregator to access whatever tools it needs — web search, document stores, APIs — without having its own MCP server configurations.

## How It All Fits Together

Here's the flow in a realistic scenario:

1. A user asks their **RockBot** agent to find a good time to meet with a client next week and prepare background information on the client's recent projects.
2. RockBot checks availability by invoking calendar tools through **mcp-aggregator**, which routes to **calendar-mcp**. Calendar-mcp checks the user's work calendar, personal calendar, and the client's shared calendar.
3. RockBot delegates the background research to a **researchagent**, discovered through the **agentregistry**. The registry knows a research agent is available (or triggers one to spin up via KEDA).
4. The researchagent uses **mcp-aggregator** to access web search tools, the company's internal knowledge base, and the CRM — all without having direct credentials to any of them.
5. Results flow back through the message bus. RockBot synthesizes the calendar availability and research results, and presents the user with proposed meeting times and a briefing document.

No single project does all of this. But together, they provide the complete infrastructure: an agent runtime with proper isolation, centralized tool access, real-world calendar integration, dynamic agent discovery, and specialized agent delegation.

## Why This Matters

The AI agent ecosystem is still young, and most frameworks treat deployment as an afterthought. They work great on a developer's laptop but don't have answers for multi-tenant environments, dynamic scaling, credential management, or inter-agent coordination in production.

That's the gap I'm trying to close. Not by building one monolithic framework that does everything, but by building focused components that follow established distributed systems principles and compose together naturally.

These projects are all open source under the MIT license:

- [RockBot](https://github.com/MarimerLLC/rockbot)
- [mcp-aggregator](https://github.com/MarimerLLC/mcp-aggregator)
- [calendar-mcp](https://github.com/MarimerLLC/calendar-mcp)
- [agentregistry](https://github.com/MarimerLLC/agentregistry)

If you're thinking about building agentic systems that need to run in real production environments, I'd love for you to take a look. Open an issue, ask questions, or contribute. The pieces are in place — now it's about making them better.
