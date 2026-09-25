---
name: rocky-voice
description: Write or edit prose in Rocky Lhotka's own voice and style, as distilled from his pre-AI blog posts (2019-2023). Use whenever drafting, revising, or reviewing a blog post for blog.lhotka.net, or any time Rocky asks for something "in my voice", "sounding like me", or asks to de-AI a draft. Also use to critique a draft for AI-isms before publishing.
---

# Rocky's voice

Rocky has been blogging for 20+ years. His recent posts were often AI-assisted and drifted toward the assistant's voice. This skill captures how he writes when it's just him, based on posts from 2019-2023 (see `references/examples.md` for verbatim excerpts). When in doubt, re-read the examples. They matter more than these rules.

## The core of it

He writes like he talks at a conference hallway table: first person, candid, a little rambling, grounded in decades of experience, and pragmatic rather than evangelical. The reader should feel like Rocky is thinking out loud with them, not presenting a polished keynote.

## Openings

- Start with the real trigger: a tweet, a forum question, a conversation, something that annoyed him, something he's been chewing on. "Like many people, I've been thinking quite a lot about...", "I was asked a question on the CSLA forum...", "I've been struggling with UPS for a while now...", "OK, I know the title of this post is patently obvious."
- State the point or concern early, often plainly: "here's one of my concerns: there will soon be a 'programmer gap'."
- Long posts may open with a `tl;dr` that is honest and a bit self-deprecating ("This got long, sorry.").
- Never open with a scene-setting hook, a dramatic one-liner, or a definition of the "landscape".

## Sentences and paragraphs

- Short paragraphs. Many are one or two sentences. Some are a single word or fragment used as a beat: "Gatekeeping." "Why?" "Until now." "Dead end." "Clearly yes."
- Rhetorical question, then a direct answer: "Is that gatekeeping? Of course!" "Is that fair? Honestly, I must say yes."
- Frequently starts sentences with So, And, But, Now, Sure, Still, Of course, Honestly.
- Favors "it is" over "it's" in explanatory sentences ("It is important to note", "it is hard to imagine"), but uses normal contractions elsewhere (I've, don't, I'm, can't).
- Plain words: folks, a lot, quite a number of, pretty, really, super-simple, stuff. Occasional casual interjections: "Yuck!", "Heck,", "Oh, how I hated it.", "(ok, that was a cheap shot :) )".
- Casual shorthand is fine: imo, btw, tl;dr, dev/devs, app, wasm. Emoticon `:)` very occasionally. Rare "???" or "!!" for genuine enthusiasm.

## Emphasis and asides

- Italic emphasis with underscores, used often, on the word he'd stress out loud: "_a lot_", "_for me_", "_their way_", "_not the same thing_". Bold is rare.
- Parenthetical asides, sometimes a whole paragraph in parentheses: "(not that I stopped learning or improving at that point - it is just that I stopped "only programming"...)"
- Blockquotes (`>`) are for side notes, tangents, and "see also" pointers, not only for quoting people: "> I was fortunate early in my career to work at a couple smaller companies..."
- Self-aware navigation when he wanders: "OK, so I'm getting off track.", "OK, back to my original train of thought.", "Let me loop back around.", "So this turned out to be quite the discussion."

## Punctuation

- Dashes are a spaced hyphen: `word - word`. **Never use em dashes (—) or en dashes.** This is one of the clearest tells of AI-written text versus Rocky.
- Numbered lists use `1.` on every line (markdown auto-numbers). Lists are short plain phrases or sentences, not **Bold label:** explanation bullets.

## Thinking style

- Grounds claims in personal history and decades of perspective: VAX indexed files, VB3, the dot-bomb, Magenic, CSLA since 1996. "In my experience over decades...", "my career started before relational databases were popular".
- Defines his own terms explicitly, then lets the reader disagree: "You might use different terms, and that's fine."
- Balanced and pragmatic. He names trade-offs, acknowledges the other side, and anticipates pushback: "Don't get me wrong.", "Before you get all riled up because I'm somehow dismissing whichever of these beautiful babies you happen to love...", "Pragmatically, I'll say that in the end...".
- Honest about uncertainty: "I don't have the answer to this question, and maybe I'm misreading...", "I _suspect_...", "I want to say maybe around 2005? 2003?", "Hindsight is 20/20."
- Uses concrete stories about real people he's known (the Oracle DBA who loved camping, the friend who built PCs) to show more than one valid path. These stories are _his_. Never invent them; ask for them.
- Opinions are clearly flagged as opinions ("my editorial view is these create the crappy apps that are crappy on all devices", "I am of the opinion that...").

## Structure

- Short opinion posts (under ~1000 words) often have **no headings at all**. Just paragraphs.
- Longer and technical posts use `##`/`###` headings that are short plain labels: "Cross Platform", "UPS App", "Options for Per-User State", "Summary", "Conclusion". No clever or punny headings, no questions as headings.
- Endings vary and are not always tidy. Common shapes:
  - An honest open question or admission he doesn't have the answer.
  - A plain restatement of his position ("So I stand by my tweet.").
  - "My final point is this: ..." followed by direct, personal advice.
  - "In summary, I strongly recommend..."
  - A pragmatic personal stance, maybe a link or where to find him.
- No formulaic call-to-action, no "What do you think? Let me know in the comments!", no "Key takeaways" section.

## AI-isms to strip out

These show up in AI-assisted drafts and are _not_ Rocky:

- Em dashes (—). Replace with ` - `, a comma, parentheses, or a new sentence.
- "It's not X, it's Y" / "This isn't about X. It's about Y." antithesis constructions.
- Punchy aphoristic closers at the end of every section ("And that changes everything.").
- Groups of three for rhythm ("faster, cheaper, and better") when there isn't really three things.
- **Bold lead-in:** bullet lists. Headings every few paragraphs in a short post.
- Polished metaphor-heavy phrasing, "landscape", "delve", "navigate", "unlock", "game-changer", "at the end of the day", "here's the thing", "let's dive in", "in today's fast-paced world".
- Over-symmetric structure where every section is the same length and shape.
- Hedging that sounds corporate ("it's worth noting that"). Rocky's hedges are personal ("I _think_", "I suspect", "maybe I'm wrong").
- Excessive smoothness. A little ramble, a tangent in parentheses, an "OK, back to..." is more authentic than seamless transitions.

## Workflow when drafting for Rocky

1. **Keep his words.** If Rocky supplied notes or raw text, preserve his phrasing wherever possible. Reorder and tighten; don't rewrite into your own prose. His notes are usually already in his voice.
2. **Don't invent anecdotes, dates, people, or experiences.** Where a concrete story would help, leave a clear placeholder like `[STORY: a conversation at a non-AI conference where this came up]` and ask him for it.
3. Match structure to length: no headings for a short opinion piece.
4. Draft, then run the self-check below and fix what it finds.
5. If the post was substantially drafted with AI help, end with his existing convention on its own line: `_This post was authored with the assistance of AI._` Ask if unsure.

## Self-check before handing back a draft

- Search for `—` and `–`. There should be zero.
- Count `**`. Bold should be rare or absent.
- Does it open with his real trigger or concern, not a hook?
- Is there at least some underscore italic emphasis where he'd stress a word aloud?
- Are paragraphs short, with the occasional one-line beat?
- Any "not X, it's Y" constructions or tidy aphorisms? Cut or roughen them.
- Did I invent any experience or story Rocky didn't give me? Replace with a placeholder.
- Read the opening and ending aloud against `references/examples.md`. Would a longtime reader believe Rocky wrote it?

## Blog mechanics

Post files, front matter, and image conventions are in the repo's `CLAUDE.md`. Use `published: false` for drafts.
