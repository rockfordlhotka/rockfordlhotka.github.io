---
layout: post
title: Agent Context Needs Timing
postDate: 2026-04-22T08:00:00-05:00
categories: []
tags: [ai, agents, mcp]
published: true
permalink:
image: /assets/2026-04-22-Agent-Context-Needs-Timing/featured-image.png
---

![Agent Context Needs Timing](/assets/2026-04-22-Agent-Context-Needs-Timing/featured-image.png)

I've been spending a lot of time thinking about how agents actually work under the hood, and one of the key insights I keep coming back to is this: _the right context, injected at the right time, is what makes an agent useful_. Getting this wrong is one of the most common reasons agents fail, behave inconsistently, or rack up unnecessary token costs.

## Defining an Agent

Let me start with how I define an agent:

> **Agent = Harness + LLM + Directive**

- The **LLM** is the language model doing the reasoning.
- The **Directive** is the system prompt, instructions, or persona that shapes how the LLM behaves.
- The **Harness** is the code that orchestrates everything — it manages the conversation loop, calls tools on behalf of the LLM, injects context, and routes the final result back to the caller.

Most of the interesting complexity lives in the Harness.

## The Harness-LLM Loop

In most agent implementations, the Harness drives a loop where it calls the LLM, handles any tool calls the LLM requests, and feeds the results back in. Here's the high-level flow:

1. **Harness calls the LLM** with the user prompt (plus the list of available tools and any other context).
2. **LLM returns a result** — this may be a final answer, or it may include one or more tool calls embedded in the response.
3. **Harness executes the tool calls** and returns the results back to the LLM as a new call, along with the full conversational history up to that point.
4. **LLM returns another result** — again, this may include more tool calls or may be a final answer.
5. **This repeats** until the LLM produces a final answer with no more tool calls, or until the Harness hits a configured iteration limit.
6. **Harness relays the final result** back to the original caller.

Each iteration through this loop, from user prompt to final result, is often called a "turn."

One critical detail at step 3: when tool calls fail, the Harness should return detailed failure information — not just an error code, but a human-readable explanation of _why_ it failed. The LLM uses that information to decide whether to retry, try a different approach, or surface the problem to the user.

## The Harness Must Provide Context at the Right Time

The Harness needs to inject the right context at each step, and the timing matters:

- **At the start of a turn**, the Harness must provide the LLM with the list of available tools _plus_ the user prompt. Without the tool list, the LLM doesn't know what capabilities it has. Without the user prompt, it doesn't know what it's trying to accomplish.
- **At each subsequent call**, the Harness must provide the tool list again, the results of the tools just called (successes _and_ failures), _plus_ the full conversational history leading up to that point. The LLM doesn't retain memory between calls — all relevant history has to be in the prompt.

Miss either of these, and the LLM either makes things up or gets confused about what it was doing.

## Context Optimization Is Hard

Here's where things get genuinely difficult: the Harness can't always predict what tools the LLM will want to call, yet it also needs to minimize the amount of context it passes on each call. Token limits are real, and bloated prompts slow things down and cost more.

The tension looks like this:

- **Pass too little per-tool detail** and the LLM will call the wrong tool — it'll pick the closest-sounding one and hope for the best.
- **Pass ambiguous or incomplete parameter schemas** and the LLM will hallucinate details to fill in the gaps. This leads to calls that look syntactically correct but are semantically wrong, and these failures can be hard to diagnose.
- **Pass too much** and you hit token limits or slow everything down with unnecessary context.

The Harness needs to provide enough tool descriptions, parameter schemas, and example values that the LLM can confidently construct a valid call — but no more than that.

Sometimes this isn't achievable in a single pass. The LLM may need to make _preliminary_ tool calls just to discover what other tools exist or what valid parameter values look like. This means extra round-trips before it can structure the _actual_ tool call it was trying to make. This is expected behavior, not a failure — but a well-designed Harness and tool set can minimize how often it's necessary.

## MCP Aggregators and the Context Trade-off

This is one of the reasons MCP gateways and aggregators (like my [mcp-aggregator](https://github.com/MarimerLLC/mcp-aggregator)) are useful. Instead of the Harness maintaining direct connections to many MCP servers and loading all of their tool details into every prompt, an aggregator acts as a single endpoint. The Harness exposes a small number of meta-tools — things like "list available servers" and "get tools for this server" — and the LLM makes targeted discovery calls when it needs details about a specific capability.

The trade-off is more round-trips between the LLM and the Harness. But the payoff is significantly smaller context payloads on most calls, because you're only loading the details of tools that are actually relevant to the current task.

Whether that trade-off is worth it depends on the number of tools in play and how many of them are actually needed for a typical request. For a Harness wired up to a small number of tightly scoped tools, direct connection and full context may be fine. For a Harness with access to dozens of MCP servers and hundreds of tools, aggregation usually wins.

## Why This Matters

If you're building agents — or trying to understand why an agent you're using behaves a certain way — understanding the Harness-LLM loop is essential. A lot of what looks like "the AI being confused" is actually the Harness not providing the right context at the right time.

The key concepts to internalize:

- **The Harness owns context management.** The LLM is stateless; every call starts fresh. The Harness is responsible for building up and injecting the history, tool lists, and results that give the LLM the information it needs.
- **Turns have structure.** Each turn starts with user input, may involve multiple LLM-Harness round-trips, and ends when the LLM produces a final answer.
- **Context injection timing determines quality.** The same LLM with the same Directive can produce dramatically different results depending on what the Harness puts in the prompt and when.

Once these concepts click, you'll have a much clearer picture of how to build agents that actually work — and how to debug them when they don't.
