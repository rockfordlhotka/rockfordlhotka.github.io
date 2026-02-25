---
layout: post
title: How RockBot Remembers
postDate: 2026-02-24T09:00:00.0000000-06:00
categories: []
tags: []
published: true
permalink:
image: /assets/2026-02-24-Agent-Memory-Systems/RockBot-thinking.png
---
Many AI models have no persistent memory. Every conversation starts fresh. They don't remember what you told them yesterday, last week, or five minutes ago in a different chat window. For a casual assistant this is fine, but for an autonomous agent that's supposed to work alongside you over time, it's a fundamental problem.

![RockBot memory](/assets/2026-02-24-Agent-Memory-Systems/RockBot-thinking.png)

I've been building an AI agent called [RockBot](https://github.com/MarimerLLC/rockbot), and one of the most interesting design challenges has been figuring out how memory should work. Not just chat history — but real memory. The kind that lets an agent know your preferences, recall context from past conversations, manage temporary scratch data, and hand off large results between sub-tasks.

What I landed on is a three-tier memory architecture. Each tier serves a distinct purpose and has different characteristics around lifetime, scope, and storage.

## Tier One: Conversation Memory

The most immediate form of memory is just tracking the current conversation. When you say "remind me what we were just discussing," the agent needs to be able to answer that without re-running everything through the model.

RockBot keeps a sliding window of the last 50 turns per session. As the conversation grows, the oldest turns are quietly dropped to keep the context manageable. Sessions that have been idle for an hour are cleaned up automatically.

This tier is intentionally ephemeral — it exists only in memory and doesn't survive a restart. That's by design. Conversation context is specific to a session and doesn't need to live forever. When you come back tomorrow and start a new session, the agent should greet you without assuming you're in the middle of the same conversation you were having yesterday.

## Tier Two: Long-Term Memory

This is where things get interesting. Long-term memory is persistent across sessions, restarts, and time. It's the agent's equivalent of actually learning things about you.

Facts are stored as individual entries organized into categories. For example:

- **user-preferences/timezone** — "User is in Chicago (America/Chicago)"
- **user-preferences/communication-style** — "Prefers concise answers; doesn't need step-by-step explanation for familiar topics"
- **project-context/rockbot** — facts about the rockbot project itself
- **anti-patterns/email** — things the agent has learned not to do when dealing with email

That last category is one of my favorites. Anti-patterns are a special class of memory that records failures — "Don't do X because Y, instead do Z." When the agent makes a mistake and gets corrected, the correction gets turned into an anti-pattern entry so the same mistake won't happen again.

### Recall at Each Turn

The tricky part of long-term memory isn't storing it — it's knowing when to surface it. You can't dump hundreds of memory entries into the model's context on every turn; that's expensive and noisy. Instead, RockBot uses BM25 ranking (the same algorithm behind many document search systems) to find the most relevant memories based on what the user is currently saying, and injects only the top results.

There's also a delta-injection mechanism: once a memory entry has been injected into the current session, it won't be surfaced again. This prevents the same facts from being repeated over and over as the conversation proceeds, and lets new, different entries surface naturally as the topic shifts.

On the very first turn of a session — before there's enough signal to do a useful keyword search — the agent falls back to injecting a small handful of random entries just to prime the context. This ensures it always has some grounding in what it knows about you, even at the start.

### Memory Enrichment

When the agent saves something to long-term memory, it doesn't just dump the raw text. A background enrichment step uses the LLM itself to expand and organize the content into well-structured, focused entries before writing them to disk. The original text might be "the user prefers early morning meetings" and the enrichment pass might produce a properly categorized, tagged entry with the right metadata. The enrichment LLM call intentionally has no access to memory tools — otherwise you could get into an infinite loop of memories creating memories.

## Tier Three: Working Memory

Working memory is the agent's scratch space. It's global, namespace-partitioned, and temporary (with configurable TTLs that default to five minutes).

The main use case is handling things that are too large or too temporary for the other tiers. A few concrete examples:

**Large tool results.** Sometimes a tool call returns a massive payload — a web page, an API response with hundreds of items, a long email thread. Stuffing that into the conversation context would eat up tokens and slow everything down. Instead, the agent saves it to working memory and just keeps a reference. Later turns can retrieve it on demand.

**Subagent data handoff.** RockBot supports spawning subagents to handle complex research or multi-step tasks in parallel. When a subagent finishes, it may have produced a large result. Rather than trying to return that in a single message, the subagent writes its output to its own working memory namespace (`subagent/{taskId}/result`) and the primary agent is told where to find it. The primary agent then retrieves only what it needs, when it needs it.

**Patrol task findings.** RockBot has autonomous "patrol" tasks that run in the background on a schedule — checking email, monitoring systems, that sort of thing. When a patrol finds something worth flagging, it writes to working memory under `patrol/{taskName}/`. The primary agent, when handling a user message, can see a summary of what patrol tasks have stored without loading all that content upfront.

### Namespace Design

The namespace scheme for working memory is intentional:

| Context | Namespace |
|---|---|
| Current user session | `session/{sessionId}` |
| Subagent task | `subagent/{taskId}` |
| Autonomous patrol | `patrol/{taskName}` |

Agents can read across namespaces — a user session can retrieve a subagent's output, or check what a patrol task has stored. But writes always go to the caller's own namespace. This keeps things organized and prevents different contexts from stomping on each other's data.

At the start of each turn, the agent gets a lightweight inventory of what's in its working memory — just keys, TTLs, and categories, not the actual content. This gives it awareness of what's available without paying the token cost of loading everything.

## The Dream Service

There's one more piece I want to mention: a background service I call the "dream pass," which runs every few hours when the agent is otherwise idle.

Its job is to autonomously maintain the long-term memory. It looks for duplicate or near-duplicate entries and merges them, refines categories, deletes stale or superseded facts, and mines the conversation log for recurring patterns that should become permanent preferences. If you've corrected the agent on the same thing three times, the dream pass will notice that pattern and create a persistent preference entry so the agent stops making that mistake.

The preference inference is sentiment-weighted: a strong correction leads to a preference being saved after one occurrence, a casual suggestion requires several instances before it rises to the level of something worth persisting.

## Why Three Tiers?

Each tier solves a different problem:

- **Conversation memory** is about coherence within a session — making the agent feel like it's paying attention.
- **Long-term memory** is about continuity across time — making the agent feel like it actually knows you.
- **Working memory** is about practical capability — giving the agent a place to handle data that doesn't fit neatly into a conversation.

Together they let RockBot behave in ways that feel qualitatively different from a stateless chat assistant. It remembers your preferences without being told them again. It can handle large, complex tasks without choking on data. It can delegate work to subagents and retrieve results cleanly. And it gets a little bit better, in small ways, every day.

Building memory into an AI agent is one of those problems that looks simple at first — just save some chat history, right? — and turns into something much more nuanced once you start taking it seriously. I think the three-tier approach hits a good balance between capability and simplicity, and I'm happy with how it's shaping up.
