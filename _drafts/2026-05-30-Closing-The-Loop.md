---
layout: post
title: Closing the Loop
postDate: 2026-05-30T08:00:00-05:00
categories: []
tags: [ai, agents, claude-code]
published: true
permalink:
image: /assets/2026-05-30-Closing-The-Loop/featured-image.png
---

![Closing the Loop](/assets/2026-05-30-Closing-The-Loop/featured-image.png)

The real performance multiplier with a coding agent isn't how fast it can write code. It's whether it can verify its own work.

When an agent implements a feature or bug fix and then immediately runs tests, checks logs, and validates behavior — all without waiting for a human — the iteration cycle collapses from hours to minutes. That's the loop I care about closing. And whether or not it closes is largely determined by decisions made _before_ the agent writes a single line of code.

## Start with a Spec, Not a Vibe

Vague prompts produce vague results. "Make the login page better" isn't a spec. It's a wish. The agent will produce _something_, but neither of you will have a clear way to know if it did the right thing.

What works is investing time upfront — with the agent, with teammates, with stakeholders — to define success. A spec doesn't need to be a formal document. It can be a set of acceptance criteria, a failing test, a user story with clear done conditions, or even a detailed conversation that gets recorded as a markdown file in the repo. What matters is that both the agent and the developer have a shared, explicit definition of what "done" means.

That definition is what the inner loop runs against. Without it, you can't close the loop at all.

## The Inner Loop

Once the spec exists, the agent's job is to implement against it and then _verify_ that it worked. This is the inner loop:

1. Implement the change.
2. Build and compile.
3. Run unit tests.
4. Run integration tests.
5. Check logs and telemetry.
6. If it worked, done. If not, understand why and go back to step 1.

For a human developer, this loop takes time at every step — context-switching, reading output, reasoning about what went wrong. For a coding agent, steps 2 through 5 can be nearly instantaneous if the right infrastructure is in place. The agent doesn't get bored, doesn't lose context, and can iterate through this loop far faster than any human.

The bottleneck isn't the LLM's ability to reason. It's whether the agent has the _tools_ to run the loop at all.

## What Closes the Loop

Three things make the inner loop possible for an agent:

**A CLI interface.** The agent needs to be able to trigger behavior from the command line. If the only way to test a feature is to click through a UI, the agent is stuck. A CLI that exercises the same code paths as the UI — seeding data, triggering workflows, querying state — gives the agent a programmable handle on the system. This is worth building deliberately, not as an afterthought.

**Logs.** Structured, readable logs that tell the agent what actually happened when it ran something. Not just "error occurred" but the full context: what was called, what failed, what state the system was in. If the agent can't read the output of its own tests and understand why something failed, it has to guess — and guessing wastes iterations.

**OTEL and observability.** For distributed systems especially, logs from one service aren't enough. OpenTelemetry traces that span service boundaries let the agent follow a request end-to-end and identify _where_ things went wrong, not just _that_ they went wrong. An agent with access to traces can diagnose integration failures that would take a human developer significant time to even reproduce.

These three things — CLI, logs, OTEL — are the difference between an agent that can close its own loop and one that has to stop and ask a human every time it needs to know if something worked.

## Iterations Are Normal

Even with all of this in place, the agent probably won't get it right on the first try. That's fine. That's expected. Human developers don't get it right the first time either — the difference is that a developer might spend thirty minutes reading a stack trace and reasoning about a fix before trying again. An agent with good observability can do the same analysis in seconds.

The value isn't that the agent is smarter. It's that the feedback loop is so much tighter. A human developer running five iterations across a workday is doing well. An agent running five iterations in fifteen minutes — because it can build, test, observe, and adjust without leaving the terminal — is a qualitatively different kind of productivity.

## The Human Outer Loop

None of this eliminates the need for humans in the process. It redefines _where_ humans belong.

The inner loop — build, test, observe, iterate — should be as automated as possible. That's where the agent earns its keep. But once the agent believes it has met the spec, a human needs to walk through the real UI and validate the actual experience. Not because the tests were wrong, but because tests don't fully capture intent. A human can tell immediately if something feels off in a way that no test suite reliably catches.

Think of it as two nested loops. The inner loop is fast, automated, and owned by the agent. The outer loop is slower, human-driven, and focused on intent and experience. The goal is to make the outer loop rare — you only hit it when the inner loop is satisfied — and to make it consequential when you do.

When the agent does its job, by the time a human reviews the feature, the rough edges are gone. The test cases pass, the logs are clean, the integration works. What's left is the genuinely human judgment: does this actually solve the problem we were trying to solve?

That's the question worth saving human attention for.
