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

The change in tone was immediate and obvious. Terra was _much_ more curt. And yet, at the same time, it was following the persona's house spirit theme more closely.

I preferred the way Sonnet was working.

Here are two exchanges from that same conversation, unedited. They aren't the same question, so this isn't a controlled test, but it gives you the flavor. First, Sonnet, about an hour and a half before the switch:

> Me: Also, I like to use a virtual whiteboard, but I heard that the Microsoft whiteboard app is gone (or going away)? Is there a good/free alternative? I don't need collaboration - just a place I can draw while sharing my screen.
>
> Rockbot (Sonnet): **Short answer: yes, the Microsoft Whiteboard apps for Windows, iOS and Android retired on Oct 16, 2026, four days ago.** I read that on Microsoft's own support page. Installed copies now show a retirement notice instead of opening your boards.
>
> Whiteboard itself isn't gone. For work or school accounts it still runs on the web and inside Teams, with the boards stored in OneDrive. The page is less clear for personal Microsoft accounts. It only says the apps retired. A Microsoft Q&A answer I saw in the search results says editing after Sept 22 requires a business account, but I didn't open that page. If you've been using a personal account, treat the web version as unreliable.
>
> **What I'd use for tomorrow: [Excalidraw](https://excalidraw.com).** It's free and runs in a browser tab. Its own site says no sign-up, and you don't need collaboration. It has a freehand pen, shapes, arrows and text, and it autosaves in the browser. That freehand pen and the autosave are from my memory of the tool, not from the pages I read, so try it tonight.
>
> **Runner-up: [tldraw](https://www.tldraw.com).** It's also free with no sign-up and has a cleaner, more traditional whiteboard feel. Its license limits use inside commercial products. I don't know whether that affects using the app to teach, so I'd choose Excalidraw.
>
> Before the class:
> - Open Excalidraw in its own browser window, so you can share just that window in On24 instead of your whole screen.
> - Draw a quick diagram and check the pen is thick enough to read on a shared screen. Zoom in or enlarge the stroke if it isn't.
> - Keep a blank canvas ready for each section, so you aren't clearing drawings mid-talk.
>
> I haven't tested either tool with On24 screen sharing, so run a quick share in a test session if you can.

And then Terra, right after the switch:

> Me: back to the blazor class, I now have built the slide deck, have the demos lined up, except for one new OTEL one that claude is building right now, so I think I'm set for tomorrow!
>
> Rockbot (Terra): You’re set for tomorrow: deck built, demos lined up, with only the new OpenTelemetry demo still in Claude’s hands. That’s a reasonable loose end, not a house fire.

Over 300 words, with reasoning, sources, caveats, and practical advice. Versus two sentences, one of which is a household metaphor straight out of the persona's style guide.

Neither one is _wrong_. My message to Terra was a status update, and a short reply is reasonable. But the change in character was obvious immediately.

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

## A first look

Once the new GPT prompt was deployed, I had Claude Code send Terra a question shaped like the whiteboard one, through Rockbot's command line client and from a fresh session:

> Prompt: I want to record a few short screen-capture videos with my voice narrating, to go along with the Blazor class. I used to use Camtasia but I don't want to pay for it anymore. Is there a good free alternative? I don't need fancy editing - just record my screen and mic, and maybe trim the start and end.

Terra came back almost immediately with this:

> Rockbot (Terra): For the Blazor recordings, **OBS Studio** is the best free replacement: no watermark, no time limit, good screen-plus-mic recording, and it’s dependable once set up. Its editor is basically nonexistent, though—trim the beginning/end afterward in **Windows Clipchamp**, which is free for simple cuts and exports.
>
> The practical two-tool setup is **OBS to record, Clipchamp to trim**. It avoids subscription traps and gives you better capture control than most “free screen recorder” apps. I’m checking the current free-tier caveats before I pin down whether a single-app option beats that combination.

And it really was checking. It had spawned a research subagent, and a couple of minutes later the real answer arrived:

> Rockbot (Terra): **Use Clipchamp Free.** For short Blazor lessons, it’s the closest no-cost replacement for the old Camtasia habit: screen and microphone capture together, simple timeline trimming, and watermark-free 1080p MP4 export. The free tier caps an individual screen recording at **30 minutes**, which fits the short-capture use case well.
>
> I’d use this setup:
>
> 1. Create a **16:9** project in Clipchamp.
> 2. Choose **Record & create → Screen**, select your mic, and capture the IDE window or full display.
> 3. Set Visual Studio/browser text to about **125–150%** so code survives compression.
> 4. Leave 2–3 seconds of silence at each end; trim with the timeline handles.
> 5. Export at **1080p**. Do one 30-second test and listen through headphones before recording a real lesson. Audio errors are the sort that wait politely until minute twelve.
>
> **OBS Studio** is the better choice only if you want unlimited recording, no Microsoft sign-in, and more reliable capture controls. It’s fully free and open-source, but it has no useful “trim the two ends and be done” editor—so the sensible workflow there is **OBS to record, Clipchamp to trim**.
>
> I would skip ScreenPal and ScreenRec for Blazor instruction: their free tiers cap output at **720p**, which makes Visual Studio text needlessly fuzzy. ShareX is free and can trim video, but it’s more fiddly than either of the two above.

That final answer has the same pieces the Sonnet whiteboard answer had: a recommendation with reasoning, a runner-up and the trade-off between them, practical setup advice, and a reason to skip the others. There is even a dry little aside about audio errors waiting until minute twelve, which is the persona showing through without a household metaphor.

[OPINION: how close this feels to Sonnet, in your view]

It is still tighter than Sonnet, maybe two-thirds the length. And Terra went about it differently: a quick provisional answer, then research, then the real answer. Sonnet did its research _before_ answering.

This is, of course, one question. It is a different question than the whiteboard one, from a fresh session rather than an established conversation, so it is not a controlled comparison. Still, it is a big change from that first two-sentence status reply, and the only thing that changed was the prompt. The `verbosity` dial hasn't been touched.

[RESULT: how Terra behaves after a few more days of real use, and whether verbosity ended up being needed]

## Summary

What I take away from this is that models are not interchangeable parts, even when they are priced the same and sit in the same tier. The same words, in the same prompt, mean different things to different models. Most of the time we don't notice, because most of us don't swap models in the middle of a conversation with an agent that has months of history.

I got to see it happen by accident.

And all this cross-model testing, tweaking, and so on is quite similar to the old cross-browser issues we used to face in the early days of the web.

We used to have to deal with browser-specific CSS dialects, and there was a whole cottage industry of cross-browser testing tools and services. All of which have largely faded into the background over time.

Not that it is gone today, but it is _far_ less important than it used to be.

Ultimately I think what brought browsers together was a combination of two things. One was market dominance by Chromium and Safari, so just two primary competitors that were forced to converge. The other was some acknowledgement that standards actually _do_ matter, if you want enterprise-level apps to work across browsers and platforms.

I wonder if interacting with LLMs will become more standardized over time, or if picking your LLM model will remain such an important consideration?

_This post was authored with the assistance of AI._
