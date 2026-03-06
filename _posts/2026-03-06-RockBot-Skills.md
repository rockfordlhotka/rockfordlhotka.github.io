---
layout: post
title: How RockBot Learns New Skills
postDate: 2026-03-06T09:00:00-06:00
categories: []
tags: [ai, agents, rockbot, dotnet]
published: true
permalink:
image: /assets/2026-03-06-RockBot-Skills/rockbot-skills.png
---

![RockBot Skills](/assets/2026-03-06-RockBot-Skills/rockbot-skills.png)

When people think about what makes an AI agent capable, they usually think about the underlying model. Bigger model, smarter agent. But in practice, a large chunk of an agent's usefulness comes from something much simpler: knowing _how_ to do things in your specific environment.

A general-purpose LLM knows that email exists. It does not know that your organization routes all support requests through a specific label, that the MCP server you're using has a quirk where threading works differently than expected, or that you've learned the hard way never to reply-all on a certain type of message. That kind of procedural, context-specific knowledge has to be built up over time — and it has to be surfaced at the right moment.

That's the problem [RockBot](https://github.com/MarimerLLC/rockbot) skills are designed to solve.

## What Skills Actually Are

In RockBot, a skill is just a markdown file. It has a name, some content describing how to do something, and a short auto-generated summary. Nothing exotic.

What makes skills useful is what they represent: distilled procedural knowledge. Not general facts, but specific instructions for how to accomplish tasks in a given environment. A skill might describe how to schedule a meeting across multiple calendars, how to structure a research delegation task, or how to handle a particular edge case when using an MCP server. Skills are the difference between an agent that knows email exists and one that actually knows how to handle your email.

Skills are stored on disk, organized by category, and version-controlled alongside the rest of the agent configuration. This means they are auditable, shareable, and recoverable. If an agent learns something wrong, you can correct it directly. If you want to pre-populate an agent with knowledge about your systems before it starts learning on its own, you can do that too.

## Why Skills Matter More Than You'd Expect

Most AI agent frameworks focus on tools: give the agent access to APIs, let it call them. Tools are necessary but not sufficient.

Tools tell the agent what actions are possible. Skills tell the agent how to use those actions well. And in real-world usage, the gap between those two things is enormous.

When you first give an agent access to a new MCP server — say, one that connects to your project management system — it can read the tool descriptions and probably muddle through. But it will make mistakes. It will try operations in the wrong order, misinterpret what certain fields mean, or miss subtle constraints that aren't obvious from the schema. Over time, through interaction, it should learn. The question is whether that learning sticks.

Without something like skills, it doesn't. Every session starts fresh. The agent makes the same mistakes it made last week, because it has no memory of having made them. Skills close that loop: when an agent learns something worth keeping, it writes a skill. The next time it needs to do something similar, it retrieves the relevant skill and starts from a better baseline.

## Closed Feedback Loops

There's a concept in control systems called a closed feedback loop: the output of a system feeds back into the system itself to correct and improve future behavior. An open loop system, by contrast, has no such correction mechanism — it just runs, regardless of how well or poorly it's doing.

Most AI agent systems today are open loop. The agent does things. If it does them badly, you correct it in the conversation. But that correction evaporates at the end of the session. The next conversation starts from zero.

RockBot's skill system is a mechanism for closing that loop. Feedback from users — both explicit (thumbs up or down on a response) and implicit (corrections mid-conversation) — feeds back into the agent's skill set. The agent doesn't just do things; it learns from doing them, in a way that persists.

This matters a lot in practice. The first time an agent handles a complex multi-step workflow, it will probably be clumsy. With a closed feedback loop, each subsequent attempt benefits from what was learned before. Without one, you're training the same session from scratch, every time.

## Pulling in the Right Skills at the Right Time

Having skills stored on disk is only useful if the agent retrieves the right ones at the right time. You can't just dump every skill into the context window on every turn — that would be expensive, noisy, and would push out other relevant information.

RockBot uses two mechanisms to handle this.

**Session-start injection.** At the beginning of each session, the agent receives a structured index of all available skills: names, auto-generated one-line summaries, ages, and last-used timestamps. This is injected once per session, not on every turn. The agent now knows what skills exist without having to load all their content.

**BM25 recall on each turn.** When a user message arrives, RockBot runs a BM25 keyword search against the skill store to find the most relevant skills for what's being discussed. BM25 is a well-understood retrieval algorithm — the same family of techniques behind many document search systems — that scores skills by how closely their content matches the current query.

Skills that surface through this search are injected into the context for that turn. But here's the key detail: once a skill has been injected in a session, it won't be injected again. This "delta injection" approach means the agent is always getting new information rather than repeatedly loading the same skills. As the conversation shifts topics, different skills surface naturally.

Skills can also cross-reference each other via `seeAlso` references. When one skill is retrieved, its related skills become candidates for retrieval too. This enables a kind of serendipitous discovery — the agent might not have searched for a particular skill, but because it's related to something it did search for, it surfaces and becomes available.

The result is a system where the agent has awareness of everything it knows (via the index) and efficient access to what's relevant right now (via BM25 recall and delta injection), without paying the token cost of loading everything upfront.

## How Skills Are Created

Skills are created by the agent itself, using the `SaveSkill` tool. When the agent encounters a workflow it expects to repeat, or learns something specific about an environment or integration, it writes a skill.

After saving, a background task uses the LLM to generate a concise one-line summary — fifteen words maximum — describing what the skill covers and when to use it. This summary is what appears in the skill index at session start. The agent sees it and can make a quick judgment about whether to retrieve the full skill content.

The agent can also update existing skills as its understanding improves, and delete skills that are no longer accurate or relevant. Skills are living documents, not static ones.

## How Skills Improve Over Time

Creating skills is the easy part. Keeping them accurate and useful over time is harder.

RockBot handles this through feedback-driven background processing.

**Explicit feedback.** The chat UI supports thumbs up and thumbs down on agent responses. Positive feedback reinforces the pattern — a note is appended to conversation history signaling that the approach was well-received. Negative feedback triggers something more significant: the agent re-evaluates its response with full access to its tool set, including skills, memory, and MCP integrations. It can consult existing skills, update them if they led it astray, or create new ones capturing what it should have done differently. Both types of feedback are recorded in a feedback store for later analysis.

**Anti-pattern mining.** The Dream Service — a background process that runs periodically when the agent is idle — scans accumulated correction feedback for failure patterns. When it finds them, it creates `anti-patterns/{domain}` memory entries that surface as constraints. "Don't do X because of Y; instead do Z." These anti-patterns are retrieved via the same BM25 mechanism as skills, so the agent sees them when it's about to do something it has been corrected on before.

**Skill consolidation.** The Dream Service also performs ongoing maintenance of the skill set itself. It looks for overlapping skills and merges them, prunes stale skills that haven't been used in a long time, detects clusters of related skills that suggest an abstract parent skill would be useful, and improves structurally sparse skills — ones that are too short to be genuinely useful. This consolidation happens automatically, without requiring explicit user action.

**Usage tracking.** Every time a skill is retrieved via `GetSkill`, its `LastUsedAt` timestamp is updated. This gives the Dream Service the signal it needs for staleness detection: a skill that hasn't been used in months is a candidate for pruning, especially if its content is thin. Skills that are frequently retrieved are treated as valuable and are candidates for optimization rather than pruning.

## The Effect Over Time

What you end up with is an agent that gets meaningfully better at its job as you use it. Not in a vague, hard-to-measure way, but concretely: specific workflows become more reliable, edge cases that caused problems are handled correctly, and the agent stops making the same class of mistakes it was corrected on before.

This is what it means to close the feedback loop. The agent's behavior isn't determined solely by the LLM's general capabilities — it's shaped by an accumulated layer of specific, contextual knowledge that grows and refines itself over time.

The first week with a new agent, you're correcting a lot. A month in, you're correcting much less. The skills system is the mechanism that makes that trajectory possible.

If you're interested in seeing how this works in practice, the full source is at [https://github.com/MarimerLLC/rockbot](https://github.com/MarimerLLC/rockbot). The skill-related code lives in `RockBot.Skills`, with the agent-side handling in `RockBot.Agent`. It's all open source under the MIT license.
