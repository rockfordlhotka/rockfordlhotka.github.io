---
layout: post
title: Same Prompt, Different Model
postDate: 2026-10-20T20:00:00-05:00
categories: []
tags: [ai, agents, llm]
published: false
permalink:
image:
---
I had an accidental experiment happen to me this week, and the result was striking enough that I want to write it down. I suspect not many people get such a direct comparison between two models, with _everything else_ held constant.

[Rockbot](https://rockbot.dev) is my personal agent. It runs 24x7 in my Kubernetes cluster, it has a long-lived memory, a set of skills it maintains for itself, and a persona. And it has a fallback chain for its main "balanced" model tier. The first choice is Claude Sonnet. If Sonnet is unavailable, the agent falls back to OpenAI's GPT-5.6 Terra, and then on down a list of other models.

This week I ran through my Anthropic usage limit, and it won't reset for about 12 days. So, exactly as designed, Rockbot quietly switched from Sonnet to Terra in the middle of an ongoing conversation.

The change in tone was immediate and obvious. Terra was _much_ more curt. Short answers, very little elaboration, almost no conversational back-and-forth. And yet, at the same time, it was following the persona more closely than Sonnet ever did.

I preferred the way Sonnet was working.

[EXAMPLE: a short exchange with Sonnet, then a similar exchange with Terra after the switch, so readers can see the difference for themselves]

## What stayed the same

This is the part that makes it an interesting comparison. Nothing changed except the model.

1. Same system prompt (which is large - around 77,000 characters once everything is assembled).
2. Same persona and style guide.
3. Same tools, same memory, same skills.
4. Same conversation history, because the switch happened mid-conversation.

Usually when people compare models they do it with a fresh chat and a toy prompt. Here it was the same agent, in the same conversation, with the same months of accumulated context. Only the LLM underneath changed.

## Is Terra just a lesser model?

My first thought was that maybe I had the tiers wrong. Perhaps Terra isn't really the equivalent of Sonnet, and I should be using a bigger GPT model to get similar results.

That turns out not to be the case, at least by price. Terra and Sonnet are priced almost the same (roughly $2 per million input tokens for both, $12 vs $10 per million output tokens). The next GPT tier up, Sol, is priced more like Claude Opus. So Terra really is the peer of Sonnet, and moving up a tier would cost about 2.5x without, I _suspect_, changing the tone very much.

> I say "I suspect" because I haven't run that experiment. My expectation is based on the cause, which I'll get to next.

## The persona

Rockbot's persona is a zashiki-warashi, a household spirit from Japanese folklore. Specifically, an _adult_ one. The style guide describes it as "long-dwelling, competent, a little wry", and it has quite a bit to say about voice and cadence. Including these lines:

> Lead with the result. The user wants the outcome, not the journey.
>
> One- or two-sentence answers are often the right answer. Resist the urge to expand.

And also this one:

> Confident, not stiff. You can be direct without being clipped. A short observation or a small aside is fine when it serves the user; padding is not.

Rockbot has been running on Sonnet, and Sonnet treats all this as a description of a _voice_. It stays in character, but it still explains its reasoning, adds context, makes the occasional dry aside, and generally carries on a conversation.

Terra treats the same text as a _specification_. It says one or two sentences are often right, so it gives one or two sentences. It says resist the urge to expand, so it doesn't expand. That's why Terra seemed both more curt _and_ more faithful to the theme. It was doing exactly what the prompt said.

So I don't think Terra is the real culprit here. I _think_ my style guide had been tuned, without me realizing it, against Sonnet's habits. Sonnet's natural tendency is to elaborate, and the brevity rules pushed back against that tendency until the result felt right. Remove that tendency and the same rules overshoot.

I think this is something a lot of people building agents will run into. If you've only ever run your prompts against one model, you've calibrated them to that model's personality. You probably don't know where, because you've never seen it behave any other way.

## The bug I found along the way

Rockbot already has a mechanism for this. There's a folder of per-model prompt files, and there was already a GPT-specific prompt in there.

It wasn't being used.

The agent resolved its per-model behavior _once_, at startup, based on the first model in the fallback chain. With Sonnet first, that meant Sonnet's settings (which have no extra prompt at all) were used for every request, including the ones that fell back to GPT. And including every request to the low and high tiers, which run on GPT models anyway.

Even worse (well, sloppier), the usage and cost logging for a fallback call was attributed to the model the call _started_ on. So Terra's tokens were being logged, and costed, as Sonnet's.

None of this was visible as long as Sonnet was healthy. It took Sonnet going away for 12 days to shake it out.

> If you build an agent with model fallback, it is worth checking that anything model-specific in your harness follows the model that _actually_ served the request, not the one you configured first.

## The fix

The changes are in [this PR](https://github.com/MarimerLLC/rockbot/pull/731) if you are curious about the details.

First, the per-model prompt now follows the model that serves each request. Each concrete model client in the fallback chain (and in each tier) injects its own prompt, replacing whatever the startup model would have used.

Second, the GPT prompt now has a section about voice and length. In short, it tells GPT that the style guide's brevity rules are about not _padding_ the answer. They are not a word budget. After leading with the result it should still give the reasoning, the detail that changes what I'd do next, and some of the persona's character.

Third, I added support for GPT's `verbosity` parameter, which is the provider's own length dial. I measured it against Terra with a simple factual question:

1. `low` - 211 characters
1. not set - 475 characters
1. `medium` - 517 characters
1. `high` - 903 characters

So "not set" is effectively `medium`, and the dial has a big effect. I'm deliberately _not_ turning it on yet though. I want to see what the prompt change does on its own first, without two variables changing at once.

## So far

[RESULT: how Terra behaves with the new GPT prompt after a few days of real use, and whether verbosity ended up being needed]

What I take away from this is that models are not interchangeable parts, even when they are priced the same and sit in the same tier. The same words, in the same prompt, mean different things to different models. Most of the time we don't notice, because most of us don't swap models in the middle of a conversation with an agent that has months of history.

I got to see it happen by accident. I don't think it will be the last time I need to tune a prompt per model.

_This post was authored with the assistance of AI._
