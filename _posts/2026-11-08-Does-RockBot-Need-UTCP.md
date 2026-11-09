---
layout: post
title: Does RockBot Need UTCP?
postDate: 2026-11-08T12:00:00-06:00
categories: []
tags: [ai, agents, rockbot, mcp, utcp]
published: false
permalink:
image:
---

Every few months a new protocol shows up that promises to fix the problems with the protocol everyone already uses. In agent tooling, that protocol is the [Model Context Protocol](https://modelcontextprotocol.io) (MCP). One of the challengers I keep hearing about is the [Universal Tool Calling Protocol](https://www.utcp.io) (UTCP).

I build and run several things that live in this space: [RockBot](https://rockbot.dev) (my agent framework and agent), an MCP aggregator, and [AgentRegistry](https://github.com/MarimerLLC/agentregistry) (a registry where agents and tool servers can find each other). So I took a serious look at whether UTCP belongs in any of them.

The short answer: not right now. But working through it did inspire an improvement to RockBot.

## What UTCP is

UTCP comes from a simple observation. Many tools an agent wants to use already have perfectly good interfaces: REST APIs, command line tools, WebSockets, gRPC services. MCP asks you to put a server in front of each one so the agent can talk MCP to it. UTCP asks: why not just describe the existing interface and let the agent call it directly?

So instead of a server, a UTCP provider publishes a *manual*. The manual is a JSON document that lists tools, each with:

- a name and description
- input and output JSON schemas
- a *call template* that says how to call the tool over its own interface: HTTP, server-sent events, streamable HTTP, CLI, WebSocket, a text file, or even MCP

The agent reads the manual and then calls the tool directly. There's no middleman. A provider can serve its manual at a well-known `/utcp` endpoint. A UTCP client can also point at an OpenAPI document and convert it into a manual automatically.

Credentials (API keys, basic auth, OAuth2 client credentials) appear in the manual as `${VARIABLE}` references. The client fills them in from its own configuration.

The project has Python and TypeScript SDKs, plus a [Go SDK](https://github.com/universal-tool-calling-protocol), all actively maintained. There's an MCP bridge in both directions, and a "code mode" library that I'll come back to. The core spec is still marked as a draft RFC, and the project is run by its founders rather than a foundation.

## How it compares to MCP

UTCP's own comparison frames it as "direct" versus MCP's "middleman" model. That's fair as far as it goes, but there's more to it than that.

**What UTCP does well:**

- No wrapper server to build, deploy and keep running for every API you want to expose.
- No extra network hop.
- You can describe an API you don't own, as long as it has a usable interface (or an OpenAPI document).

**What MCP has that UTCP doesn't:**

- **Everything that isn't a tool call.** MCP has prompts, resources, elicitation (the server asking the user a question mid-call), sampling, tasks and notifications. UTCP is about calling tools, full stop. My own MCP servers use prompts, OAuth sign-in and mid-call questions, so this matters to me.
- **A server-side place for secrets.** With UTCP, the client holds the credentials, and the manual decides which URL they get sent to. A manual is a list of instructions for your agent's secrets. If you load one you don't fully trust, it can send your API key to the wrong place, or, with a CLI call template, run commands on your machine. The UTCP RFC acknowledges this and tells you to load only trusted manuals. That's reasonable advice, and it's also the problem.
- **The ecosystem.** MCP is everywhere. UTCP manuals are rare.

**The middleman has value.** UTCP treats the server in the middle as overhead to get rid of, but we've learned this lesson before. Back when client/server apps talked straight to the database, every client machine needed a connection string, and often the credentials to go with it. Moving to n-tier, with an application server between the clients and the database, meant the keys to the database lived only on the app server. No client ever saw them, and the whole system was more secure as a result. An MCP server (or an aggregator in front of several) plays the same role for agents. The agent holds a credential for the MCP server, not the keys to every API behind it. A central server is also the natural place for logging and tracing. Every tool call passes through one spot, so you can record who called what, with which arguments and what result, and connect it all with distributed tracing. When the agent calls tools directly, that record is scattered across every client and every API, if it exists at all.

**MCP has also been closing the gap.** A big part of UTCP's original pitch was that MCP is heavy and stateful. The [2026-07-28 MCP specification](https://blog.modelcontextprotocol.io/posts/2026-07-28-release-candidate/) made the protocol stateless: no sessions, no initialization handshake, each request stands on its own, and any request can land on any server instance behind a plain load balancer. That knocks out a lot of the operational argument for skipping MCP.

## RockBot

RockBot's tool system isn't tied to MCP. At the core there's a tool registry: each tool is a name, a description, a JSON schema and an executor. MCP servers, A2A agents, web tools, scripts and wisps all register into it. Adding a UTCP tool source would be easy at the basic level.

But I don't think it should, for three reasons.

**RockBot already solves the problems UTCP is pitching.** A lot of the "MCP doesn't scale" argument is really about large tool sets: too many tools for a model to choose from, and too much context spent describing them. RockBot deals with that by loading tools only when they're needed:

- the model searches for tools and activates them per session
- each tool gets its own typed definition, not a generic "call some tool" shim
- markdown guides for each subsystem
- caps that keep each request under the provider's tool limit

UTCP's default tool search is a simple keyword match on tags and descriptions. Getting UTCP tools into all of RockBot's machinery would be real work, and it would make things worse for the tools I actually use.

**Security.** RockBot follows a "nothing trusts the LLM" design, and the model can manage some of its own server connections. Letting the model add a UTCP manual would mean letting it decide where my secrets go. That's not a door I want to open.

**I've been here before.** RockBot used to have a REST API gateway, and I removed it. That was essentially the same idea as UTCP, under a different name.

## The MCP aggregator

The aggregator is different. Its job is to sit in front of many backend servers and present them as one MCP server, with tool search, typed wrapper tools and skills. A UTCP or OpenAPI backend would fit there as a gateway: read the manual (or OpenAPI document), present those tools as MCP tools, and make the calls from the server side.

That flips UTCP's model around, and gets back the value of a server in the middle. The credentials stay on the server, where I can control them, every call is logged and traced in one place, and every MCP client gets the tools without knowing UTCP exists. It's the opposite of UTCP's "no middleman" philosophy, and I'm fine with that.

If I do this, it'll be **OpenAPI import first**, because OpenAPI documents are everywhere and UTCP manuals aren't. I'd also accept only HTTP-style call templates (no CLI), and only from servers an operator registers. I'll do it when I have a specific REST API I want to expose, and not before.

## AgentRegistry

This is the most natural fit of the three. AgentRegistry is deliberately protocol agnostic. It already lists A2A agents, MCP servers and ACP agents. It stores each one's native description as metadata, and it doesn't speak any of those protocols itself. Adding UTCP would mean one more protocol type and a small adapter, about the size of the ACP one.

There's also a fun twist: the registry already publishes an OpenAPI document, and UTCP clients can convert OpenAPI on the fly. So a UTCP client should be able to use the registry's discovery API today, with no changes at all.

The cautions still apply. Discovery in the registry is public. A UTCP manual is more dangerous than an agent card, because it tells clients where to send credentials and can include CLI commands. If I add UTCP support, the registry should reject CLI templates, require HTTPS, and require an admin key to register a manual.

And the honest question is: who would use it? Today, nobody in my world publishes or consumes UTCP manuals. So: a good fit, but not yet.

## Inspired by code mode

The most interesting thing I found wasn't the protocol. It was [code mode](https://github.com/universal-tool-calling-protocol/code-mode). Instead of the model making one tool call at a time, with a full round trip through the LLM for each, the model writes a short script that calls tools as ordinary functions. The script runs in a sandbox, and only the final result goes back into the model's context.

This idea doesn't depend on UTCP; it works just as well over MCP tools. And it's about the same problem RockBot's *wisps* address.

A wisp is a lightweight pipeline the agent can spawn for a multi-step task when it knows the steps in advance. Steps either call a tool directly (no LLM involved at all), or hand a tightly-scoped prompt to a small, cheap model. A five-step task that might cost 60-80K tokens as a subagent costs about 2-12K as a wisp.

So do wisps already cover code mode? Mostly, but not completely.

**Where wisps are better.** The harness runs a wisp's steps, not code the model wrote. So RockBot doesn't need to sandbox model-written code, and the "nothing trusts the LLM" principle holds. Because a wisp is *data*, RockBot can:

- validate it before running anything
- limit which tools it can touch
- recognize when the model retries the same wisp, and learn from failures over time
- notice when a tool it depends on has changed

None of that works on arbitrary scripts. (UTCP's code mode sandboxes code with Node's `vm` module, which isn't a security boundary. The approach could be implemented more securely.)

**Where code mode is better: control flow that depends on the data.** A wisp is a straight list of steps. It can pass results from one step to the next and skip ahead on failure, but it can't loop or make decisions. Take "search my email, then fetch each of the N messages that came back". In code mode that's a `for` loop. In a wisp, today, it needs a step handled by the small model, so tokens are back on the meter for purely mechanical work.

Code mode made that gap obvious, and it inspired a fix that fits how wisps already work. Rather than adopting code mode itself, I've opened an issue to add a [`for_each` step to wisps](https://github.com/MarimerLLC/rockbot/issues/779). A direct step names a list from a previous step's result, and the harness runs it once per item, with limits on item count and concurrency and per-item failure handling. A simple `when` condition may follow. Both stay declarative, so wisps keep everything that comes from being data, and pick up most of what makes code mode appealing.

## Where I landed

| | Can it use UTCP? | Should it? |
|---|---|---|
| RockBot | Yes, easily | No. It already handles tool discovery better, and UTCP would add risk |
| MCP aggregator | Yes, as a gateway | Maybe, starting with OpenAPI import, when there's a real API to expose |
| AgentRegistry | Yes, most naturally | Good fit, but not until someone publishes UTCP manuals |
| Code mode | Mostly covered by wisps | Add `for_each` (and maybe `when`) to wisps instead |

UTCP has a real point: wrapping every existing API in a server just so an agent can call it is a lot of ceremony. But for me, the answer is to put that conversion in one trusted place (the aggregator) rather than push it into every agent. And the best idea in the UTCP ecosystem, having the model write code that orchestrates tool calls, doesn't depend on UTCP at all.

I'll keep an eye on it. If the spec leaves draft status, or a solid .NET SDK shows up, or people start publishing manuals for APIs I care about, I'll take another look.

_This post was authored with the assistance of AI._
