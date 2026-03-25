---
layout: post
title: "Tools and Skills: Better Together"
postDate: 2026-03-25T08:00:00-06:00
categories: []
tags: [ai, agents, rockbot, dotnet]
published: true
permalink:
image: /assets/2026-03-25-Tools-And-Skills-Better-Together/rockbot-tools-plus-skills.png
---

![Tools and Skills: Better Together](/assets/2026-03-25-Tools-And-Skills-Better-Together/rockbot-tools-plus-skills.png)

I keep running into a version of the same question when talking about AI agent design: if you have good enough skills — detailed procedural knowledge in markdown files — do you even need MCP servers and other tools?

No. You absolutely still need tools. But the question itself reveals a misunderstanding about what skills actually are, and I think it's worth unpacking.

Skills and tools are not competing approaches. You can't replace one with the other. In practice, they're deeply intertwined — and trying to pit them against each other misses the entire point of both.

I'm going to use [RockBot](https://github.com/MarimerLLC/rockbot) as my example throughout this post because it's what I'm building and I know it best, but these concepts are not specific to RockBot. Claude Code has its `CLAUDE.md` files and tool use. GitHub Copilot has instruction files, skills, and MCP integration. Cursor, Windsurf, and other AI coding agents all have some form of this pattern. The relationship between tools and skills is a fundamental design concern for any AI agent, not a feature of any one product.

## The Basics

I've written about [RockBot's tools](https://blog.lhotka.net/2026/03/09/Agent-Resources-And-Tools) and [RockBot's skills](https://blog.lhotka.net/2026/03/06/RockBot-Skills) separately, so I won't rehash everything here. The short version:

[Tools](https://blog.lhotka.net/2026/03/09/Agent-Resources-And-Tools) are functions the agent can call to take action in the world. Send an email, check a calendar, search the web, invoke an A2A agent, store a memory. Without tools, an agent can only chat. A skill file cannot send an email. A skill file cannot look up what's on your calendar. Tools are how agents _do things_.

[Skills](https://blog.lhotka.net/2026/03/06/RockBot-Skills) are markdown files that capture procedural, context-specific knowledge the agent has built up over time. They encode lessons from past failures, successful patterns, environment-specific conventions, and — critically — knowledge about _how to use tools well_.

That last point is the one people miss. Knowing that a hammer exists is different from knowing how to drive a nail without splitting the wood. The hammer is the tool. The technique is the skill. You need both.

## Tools Come with Their Own Skills

In [RockBot](https://github.com/MarimerLLC/rockbot), the relationship between tools and skills isn't just conceptual — it's built into the architecture.

Every tool subsystem in the RockBot framework can register a base-level **tool guide** when it starts up. This is a default skill that the subsystem itself provides, describing how its tools should be used. When the MCP integration subsystem loads, it registers a guide explaining how `mcp_list_services`, `mcp_get_service_details`, and `mcp_invoke_tool` work together. The A2A subsystem does the same for agent-to-agent communication. The web subsystem explains how search and browsing tools relate. Memory, scheduling, subagents — each subsystem brings its own guide.

The agent uses `list_tool_guides` and `get_tool_guide` to discover and retrieve these guides. On day one, before any learning has happened, the agent already has grounded knowledge about how to use its tools — not just what they are, but how to use them effectively.

So right from the start, tools and skills are coupled. The tools arrive with skills already attached.

## Skills Improve Through Tool Usage

Those base-level tool guides are a starting point, not a ceiling.

As the agent uses its tools across real interactions, it learns. It discovers edge cases, finds better sequences, encounters caveats that weren't obvious from the schema alone. Through RockBot's [feedback loop](https://blog.lhotka.net/2026/03/06/RockBot-Skills) — explicit thumbs up/down from users and implicit correction signals from conversations — the agent refines and extends its skills.

I have a great real-world example of this. A while back, RockBot kept [creating calendar events at the wrong time](https://blog.lhotka.net/2026/03/10/An-Agentic-Tale). It would send 4 PM Central to the calendar MCP server, and the event would show up at 11 AM. Four times in a row. It turned out the MCP server had a bug where it silently ignored the timezone parameter and treated all times as UTC.

The tool guide for the calendar MCP server didn't mention this problem — because it didn't exist when the guide was written. But after that painful debugging session, the agent learned the workaround (send UTC times directly), and that knowledge was captured as an updated skill. The next time the agent scheduled something, it didn't make the same mistake. That learning was _entirely dependent_ on having the tool in the first place. You can't learn to work around a calendar bug if you don't have a calendar.

That's the pattern. The skill describing how to use the calendar MCP server on day one is fairly generic. After weeks of actual calendar management, that skill becomes precise: how to handle recurring events, what to do when attendee time zones differ, what the server does and doesn't support. The agent has learned by doing, and the skill has grown because of it.

## Skills Do Many Things — Including Making Tools Better

I want to be clear that skills aren't _only_ about tool usage. Skills capture all sorts of procedural knowledge: how to structure a research delegation, what tone to use with different contacts, how to format reports. Many skills have nothing to do with specific tools.

But a large and important subset of skills exist specifically to make tool usage more effective. And that's the insight I think gets lost when people frame this as "tools vs. skills": skills aren't an alternative to tools. They're a _multiplier_ on tools.

Skills are operational knowledge — knowledge _about tools_, _for tools_, _refined through using tools_. They don't sit above the tool layer in the architecture. They sit right alongside it, making it work better.

## The Better Together Design

What [RockBot](https://github.com/MarimerLLC/rockbot) demonstrates is that these two concepts work in concert at every level:

**Tools provide capability.** They are the agent's connection to the real world — email, calendars, file storage, web, other agents.

**Tool guides provide starting knowledge.** Each subsystem ships with a skill that grounds the agent from the moment tools become available. The agent never has to figure out a subsystem entirely from scratch.

**Experience improves that knowledge over time.** As the agent uses tools, encounters failures, receives feedback, and discovers edge cases, skills get richer and more precise. Tool usage becomes more effective and more reliable.

Remove the tools and you have an agent that can describe how things _should_ work but can't actually do anything. Remove the skills and you have an agent that stumbles through every interaction, making the same mistakes over and over because nothing it learns ever sticks.

Together? You get an agent that keeps getting better at its job.

And again — this isn't a RockBot-specific insight. Whether you're configuring GitHub Copilot with custom instructions and MCP servers, setting up Claude Code with `CLAUDE.md` files and tool access, or building your own agent framework from scratch, the same principle applies. Tools give your agent the ability to act. Skills give it the knowledge to act _well_. Invest in both.
