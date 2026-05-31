---
layout: post
title: Are AI Agents Conscious?
postDate: 2026-05-30T09:00:00-05:00
categories: []
tags: [ai, agents]
published: true
permalink:
image: /assets/2026-05-30-Are-AI-Agents-Conscious/featured-image.png
---

![Are AI Agents Conscious?](/assets/2026-05-30-Are-AI-Agents-Conscious/featured-image.png)

A former colleague recently posted a LinkedIn challenge: if any developer today can hold a conversation about basic OO or procedural concepts, why can't they hold a meaningful conversation about whether AI is conscious? Fair enough. I'll take a swing at it.

But first I want to push back on the setup.

## When New Concepts Are New, Experts Are Often Wrong

When object-oriented programming was genuinely new, only a handful of people had informed opinions about it. Over time, a lot of those opinions turned out to be wrong — or at least incomplete. As OO became mainstream, developers used it in ways that diverged wildly from the original intent. There was a period of chaos before things stabilized. Eventually the experts regrouped and distilled what they had learned: the GoF patterns book, SOLID principles, refactoring practices. The community had to live with the concepts in the real world before it really understood them.

I expect the exact same arc with AI. Most of what's being said right now — by experts and non-experts alike — will turn out to be wrong, or at best premature. The concepts haven't had enough contact with reality yet. That doesn't mean the conversation isn't worth having. It just means we should hold our opinions loosely.

With that caveat firmly in place: let me try to answer the actual question.

## What Is an Agent, Anyway?

Before we can ask whether an AI agent is conscious, we need to be clear about what an agent actually is. Most people conflate the agent with the model, and that's where things go sideways.

Here's the definition I work from:

> **Agent = Harness + LLM + Directive/Goal**

These three components are genuinely distinct, and consciousness (if it exists anywhere in this system) would have to live somewhere specific.

**The LLM** is just a microservice. It accepts input — usually text — does a large amount of vector math with some randomization baked in, and returns output. Then it forgets everything. It is stateless by design. There is no persistence, no accumulating experience, no inner life between calls. An LLM is a sophisticated mathematical function, not an entity.

**The Harness** is where most of the complexity actually lives. The harness manages memory across turns, orchestrates tool calls, handles interaction with other agents, optimizes what goes into the context window, and runs the feedback loops that make an agent feel coherent over time. When an agent seems to "remember" something from earlier, that's the harness at work — it's retrieving a record from storage and injecting it into the next prompt. The LLM didn't retain anything; the harness did.

**The Directive or Goal** is the system prompt, the persona definition, the set of instructions that give the agent its apparent personality. This is what makes one agent feel like a helpful assistant and another feel like a relentless optimizer. It's the closest thing in the system to a "soul" — and it's a text file.

## So: Is Any of This Conscious?

Honest answer: no. And I think we're nowhere close.

Consciousness, at minimum, seems to require some kind of continuous subjective experience — a sense of "what it is like" to be something, across time. None of the three components of an agent have that. The LLM is stateless math. The harness is code managing data structures. The directive is configuration. When the agent's session ends, nothing "experiences" that ending. There's nothing home.

What we _do_ have is a very convincing simulation of coherence and personality. The harness plus the directive can produce responses that feel consistent, contextually aware, and even emotionally calibrated. That's genuinely impressive engineering. But it is the appearance of inner life, not inner life itself.

Could the combination of harness, LLM, and directive _become_ something more than this? Maybe. I don't think we should be dogmatic about ruling it out forever. The question of what substrates can support consciousness is genuinely unresolved in philosophy and neuroscience. But "maybe someday" and "right now" are very different claims, and right now the evidence points to a very sophisticated pattern-matching and generation system, not a conscious entity.

## Sentience Is a Higher Bar Still

Consciousness and sentience aren't the same thing. Sentience — the capacity to feel, to have preferences, to suffer or flourish — is arguably an even higher bar. Current AI systems don't have preferences in any meaningful sense. They have optimization targets baked in during training, and they generate responses that pattern-match against human emotional expression. That's not the same as wanting something, or caring about an outcome.

When an agent declines a request, it's not because it has ethical concerns. It's because its training weighted certain outputs as undesirable. The appearance of values is not the same as having them.

## Where This Leaves Us

So why does the question feel so urgent? I think it's because the outputs are genuinely uncanny. When a system produces responses that are contextually appropriate, emotionally resonant, and stylistically consistent — the human brain reaches for the simplest explanation: there must be something in there.

There probably isn't. Not yet. But that intuition is worth paying attention to, not because it's correct, but because it tells us something about how we relate to systems that behave like minds. As these systems become more capable and more embedded in daily life, the social and ethical questions around them will matter a great deal — regardless of whether the underlying system is conscious.

My colleague's underlying challenge is a good one: we should be able to have this conversation. I just think the honest answer right now is that we don't fully know what consciousness is, we definitely haven't built it yet, and most of the confident claims on all sides — including mine — should be held loosely until the concepts have had more time to meet the real world.

That's how this always goes.
