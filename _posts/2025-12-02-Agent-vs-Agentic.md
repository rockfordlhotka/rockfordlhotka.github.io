---
layout: post
title: Agent vs Agentic
postDate: 2025-12-02T14:39:53.3750407-05:00
categories: []
tags: []
published: true
permalink: 
featuredImageUrl: /assets/2025-12-02-Agent-vs-Agentic/agent-agentic.png
---
From Copilot:

_The term "agent" typically refers to an entity that can act independently and make decisions based on its programming or objectives. In contrast, "agentic" describes the quality or characteristic of being an agent, often emphasizing the capacity for self-directed action and autonomy._

In practice though, the term "agent" is highly overloaded. Over the years, "agent" has been used to describe software programs, types of microservice, human roles, and now some aspects of AI systems.

Less common, until now, was the term "agentic". What seems to be emerging is the idea that an "agentic" system has one or more "agents" that that are more advanced than simple "agents". Yeah, that makes everything more clear!

![AI Pragmatist](/assets/2025-12-02-Agent-vs-Agentic/agent-agentic.png)

## What is an Agent?

There are many ways people build agents today, and many types of software are called agents.

For example, inside some AI chat software, an agent might be defined as a pre-built set of prompts, and possibly access to specific tools via MCP (model context protocol). Such an agent might be focused on building a specific type of software, or finding good flights and hotels for a trip. In any case, these types of agents are built into a chat experience, and are triggered by user requests.

For a recent client project, we wrote an agent that was triggered by user requests, and routed those requests to other agents. Those other agents (subagents?) vary quite a lot in implementation and complexity. One of them, for example, was _also_ an agent that routed user requests to various APIs to find information. Another exposed a set of commands over an existing REST API. What they all have in common is that they are triggered by user requests, and they do not have any autonomy or ability to act without user input.

Sometimes people talk about an agent as being able to act autonomously. To trigger on events other than direct user requests. I think this is where the term "agentic" starts to make more sense.

## What is Agentic?

In my mind, I differentiate between agents that are triggered by user requests, and those that can act autonomously. The latter I would call "agentic". Or at least autonomous agents enable the creation of agentic systems.

And I guess that's the key here: there's a difference between simple agents that respond to user requests, and more complex agents that can act on their own, make decisions, and pursue goals without direct user input.

An agentic system has at least one, but probably many, agents that can operate autonomously. These agents can perceive their environment, make decisions based on their programming and objectives, and take actions to achieve specific goals.

This is not to say that there won't _also_ be simpler agents and tools within an agentic system. In fact, an agentic system might be composed of many different types of agents, some of which are simple and user-triggered, and others that are more complex and autonomous. And others that are just tools used by the agents, probably exposed via MCP.

## How Big is an Autonomous Agent?

Given all that, the next question to consider is the "size" or scope of an autonomous agent.

Here I think we can draw on things like the Single Responsibility Pattern, Domain Driven Design (DDD) and microservices architecture. An autonomous agent should probably have a well-defined scope or bounded context, where it has clear responsibilities and can operate independently of other agents.

I tend to think about it in terms of a human "role". Most human workers have many different roles or responsibilities as part of their job. Some of those roles are often things that could be automated with powerful end software. Others require a level of judgement to go along with any automation. Still others require empathy, creativity, or other human qualities that are hard to replicate with software.

### Building Automation

One good use of AI is to have it build software that automates specific tasks. In this case, an autonomous agent might be responsible for understanding a specific domain or task, and then generating code to automate that task. The AI is not involved in the actual task, just in understanding the task and building the automation.

This is, in my view, a good use of AI, because it leverages the strengths of AI (pattern recognition, code generation, etc.) to creat tools. The tools are almost certainly more cost effective to operate than AI itself. Not just in terms of money, but also in terms of the overall ethical concerns around AI usage (power, water, training data).

### Decision Making

As I mentioned earlier though, some roles require judgement and decision making. In these cases, an autonomous agent might be responsible for gathering information, analyzing options, and making decisions based on its programming and objectives.

This is probably done in combination of automation. So AI might be used to create automation for parts of the task that are repetitive and well-defined, while the autonomous agent focuses on the more complex aspects that require judgement.

Earlier I discussed the ambiguity around the term agent, and you can imagine how this scenario involves different types of agent:

- Simple agents that are triggered by user requests to gather information or perform specific tasks.
- Autonomous agents that can analyze the gathered information and make decisions based on predefined criteria.
- Automation tools that are created by AI to handle repetitive tasks.

What we've created here is an agentic system that leverages different types of agents and automation to achieve a specific goal. That goal is a single role or responsibility, that should have clear boundaries and scope.

## Science Fiction Inspiration

The idea of autonomous agents is not new. It has been explored by Isaac Asimov in his Robot series, where robots are designed to act autonomously and make decisions based on the Three Laws of Robotics.

More recent examples come from the works of Ian Banks, Neil Asher, Alastair Reynolds, and many others. In these stories, autonomous agents (often called AIs or Minds) are capable of complex decision making, self-improvement, and even creativity. These fictional portrayals often explore the ethical and societal implications of autonomous agents, which is an important consideration as we move towards more advanced AI systems in the real world.

Some of these authors explore utopic visions of AI, while others focus on dystopic outcomes. Both perspectives are valuable, as they highlight the potential benefits and risks associated with autonomous agents.

I think there's real value in looking at these materials for terminology that can help us better understand and communicate about the evolving landscape of AI and autonomous systems. Yes, we'll end up creating new terms because that's how language works, but a lot of the concepts like agent, sub-agent, mind, sub-mind, and more are already out there.

## Conclusion

Today the term "agent" is overloaded, overused, and ambiguous. Collectively we need to think about how to better define and communicate about different types of agents, especially as AI systems become more complex and capable.

The term "agentic" seems to be less overloaded, and is useful for describing systems that have one or more autonomous agents in the mix. These autonomous agents can perceive their environment, make decisions, and take actions to achieve specific goals.

We are at the beginning of this process, and this phase of every new technology involves chaos. It will be fun to learn lessons and see how the industry and terminology evolves over time.
