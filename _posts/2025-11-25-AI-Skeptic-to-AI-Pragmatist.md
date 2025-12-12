---
layout: post
title: AI Skeptic to AI Pragmatist
postDate: 2025-11-25T14:39:53.3750407-05:00
categories: []
tags: []
published: true
permalink: 
image:
---
A few months ago I was an AI skeptic. I was concerned that AI was similar to Blockchain, in that it was mostly hype, with little practical application outside of a few niche use cases.

I still think AI is overhyped, but having intentionally used LLMs and AI agents to help build software, I have moved from skeptic to pragmatist.

I still don't know that AI is "good" in any objective sense. It consumes a lot of water and electricity, and the training data is often sourced in ethically questionable ways. This post isn't about that, as people have written extensively on the topic.

The thing is, I have spent the past few months intentionally using AI to help me do software design and development, primarily via Copilot in VS Code and Visual Studio, but also using Cursor and a couple other AI tools.

The point of this post is to talk about my experience with actually using AI successfully, and what I've learned along the way.

In my view, as an industry and society we need to discuss the ethics of AI. That discussion needs to move past a start point that says "AI isn't useful", because it turns out that AI can be useful, if you know how to use it effectively. Therefore, the discussion needs to acknowledge that AI is a useful tool, and _then_ we can discuss the ethics of its use.

![AI Pragmatist]({{ site.url }}/assets/2025-11-25-AI-Skeptic-to-AI-Pragmatist/ai-hero.png)

## The Learning Curve

The first thing I learned is that AI is not magic. You have to learn how to use it effectively, and that takes time and effort. It is also the case that the "best practices" for using AI are evolving as we use it, so it is important to interact with others who are also using AI to learn from their experiences.

For example, I started out trying to just "vibe code" with simple prompts, expecting the AI to just do the right thing. AI is non-deterministic though, and the same prompt can generate different results each time, depending on the random seed used by the AI. It is literally a crap shoot.

To get any reasonable results, it is necessary to provide context to the AI beyond expressing a simple desire. There are various patterns for doing this. The one I've been using is this:

1. Who am I? (e.g. "I am a senior software engineer with 10 years of experience in C# and .NET.")
2. Who are you? (e.g. "You are an AI assistant that helps software engineers write high-quality code.")
3. Who is the end user? (e.g. "The end users are financial analysts who need to access market data quickly and reliably.")
4. What are we building? (e.g. "You are building a RESTful API that provides access to market data.")
5. Why are we building it? (e.g. "The API will help financial analysts make better investment decisions by providing them with real-time market data.")
6. How are we building it? (e.g. "You are using C#, .NET 8, and SQL Server to build the API.")
7. What are the constraints? (e.g. "The API must be secure, scalable, and performant.")
You may provide other context as well, but this is a good starting point. What this means is that your initial prompt for starting any work will be fairly long - at least one sentence per item above, but in many cases each item will be a paragraph or more.

Subsequent prompts in a session can be shorter, because the AI will have that context. _However_, AI context windows are limited, so it your session gets long (enough prompts and responses), you may need to re-provide context.

To that point, it is sometimes a good idea to save your context in a document, so you can reference that file in subsequent requests or sessions. This is easy to do in VS Code or Visual Studio, where you can reference files in your prompts.

## A Mindset Shift

Notice that I sometimes use the term "we" when talking to the AI. This is on purpose, because I have found that it is best to think of the AI as a collaborator, rather than a tool. This mindset shift is important, because it changes the way you interact with the AI.

> Don't get me wrong - I don't think of the AI as a person - it really _is a tool_. But it is a tool that can collaborate with you, rather than just a tool that you use.

When you think of the AI as a collaborator, you are more likely to provide it with the context it needs to do its job effectively. You are also more likely to review and refine its output, rather than just accepting it at face value.

## Rate of Change

Even in the short time I've been actively using AI, the models and tools have improved significantly. New features are being added all the time, and the capabilities of the models are expanding rapidly. If you evaluated AI a few months ago and decided it wasn't useful for a scenario, it might well be able to handle that scenario now. Or not. My point is that you can't base your opinion on a single snapshot in time, because the technology is evolving so quickly.

## Effective Use of AI

AI itself can be expensive to use. We know that it consumes a lot of water and electricity, so minimizing its use is important from an ethical standpoint. Additionally, many AI services charge based on usage, so minimizing usage is also important from a cost standpoint.

What this means to me, is that it is often best to use AI to build deterministic tools that can then be used without AI. Rather than using AI for repetative tasks during development, I often use AI to build bash scripts or other tools that can then be used to perform those tasks without AI.

Also, rather than manually typing in all my AI instructions and context over and over, I store that information in files that can be referenced by the AI (and future team members who might need to maintain the software). I do find that the AI is very helpful for building these documents, especially Claude Sonnet 4.5.

GitHub Copilot will automatically use a special file you can put in your repo's root:

```text
/.github/copilot-instructions.md
```

It now turns out that you can put numerous files in an `instructions` folder under `.github`, and Copilot will use all of them. This is great for organizing your instructions into multiple files.

This file can contain any instructions you want Copilot to use when generating code. I have found this to be very helpful for providing context to Copilot without having to type it in every time. Not per-prompt instructions, but overall project instructions. It is a great place to put the "Who am I?", "Who are you?", "Who is the end user?", "What are we building?", "Why are we building it?", "How are we building it?", and "What are the constraints?" items mentioned above.

You can also use this document to tell Copilot to use specific MCP servers, or to avoid using certain ones. This is useful if you want to ensure that your code is only generated using models that you trust.

## Prompt Rules

Feel free to use terms like "always" or "never" in your prompts. These aren't foolproof, because AI is non-deterministic, but they do help guide the AI's behavior. For example, you might say "Always use async/await for I/O operations" or "Never use dynamic types in C#". This helps the AI understand your coding standards and preferences.

Avoid being passive or unclear in your prompts. Instead of saying "It would be great if you could...", say "Please do X". This clarity helps the AI understand exactly what you want.

If you are asking a question, be explicit that you are asking a question and looking for an answer, otherwise the AI (in agent mode) might just try to build code or other assets based on your question, thinking it was a request.

GitHub Copilot allows you to put markdown files with pre-built prompts into a `prompts` folder under `.github`. You can then reference these prompts in your code comments to have Copilot use them using the standard `#` file reference syntax. This is a great way to standardize prompts across your team.

## Switch Models as Needed

You may find that different AI models work better for various purposes or tasks.

For example, I often use Claude Sonnet 4.5 for writing documentation, because it seems to produce clearer and more concise text than other models. However, I often use GPT-5-Codex for code generation, because it seems to understand code better.

> That said, I know other people who do the exact opposite, so your mileage may vary. The key is to experiment with different models and see which ones work best for your specific needs.

The point is that you don't have to stick with a single model for everything. You can switch models as needed to get the best results for your specific tasks.

In GitHub Copilot there is a cost to using premium models, with a multiplier. So (at the moment) Sonnet 4.5 is a 1x multiplier, while Haiku is a 0.33x multiplier. Some of the older and weaker models are 0x (free). So you can balance cost and quality by choosing the appropriate model for your task.

## Agent Mode vs Ask Mode

GitHub Copilot has two primary modes of operation: Agent Mode and Ask Mode. Other tools often have similar concepts.

In Ask mode the AI responds to your prompts in the chat window, and doesn't modify your code or take other actions. The vscode and Visual Studio UIs usually allow you to _apply_ the AI's response to your code, but you have to do that manually.

In Agent mode the AI can modify your code, create files, and take other actions on your behalf. This is more powerful, but also more risky, because the AI might make changes that you don't want or expect.

I'd recommend starting with Ask mode until you are comfortable with the AI's capabilities and limitations. Once you are comfortable, you can switch to Agent mode for more complex tasks. Agent mode is a _massive_ time saver as a developer!

By default, Agent mode does prompt you for confirmation in most cases, and you can disable those prompts over time to loosen the restrictions as you become more comfortable with the AI.

## Don't Trust the AI

The AI can and _will_ make mistakes. Especially if you ask it to do something complex, or look across a large codebase. For example, I asked Copilot to create a list of all the classes that implemented an interface in the CSLA .NET codebase. It got most of them, but not all of them, and it included some that didn't implement the interface. I had to manually review and correct the list.

I think it might have been better to ask the AI to give me a grep command or something that would do a search for me, rather than trying to have it do the work directly.

However, I often have the AI look at a limited set of files and it is almost always correct. For example, asking the AI for a list of properties or fields in a single class is usually accurate.

## Use Git Commit like "Save Game"

I've been a gamer for most of my life, and one thing I've learned from gaming is the concept of "save games". In many games, you can save your progress at any point, and then reload that save if you make a mistake or want to try a different approach.

This is true for working with AI as well. Before you ask the AI to make significant changes to your code, make a git commit. This way, if the AI makes changes that you don't want or expect, you can easily revert to the previous state.

I find myself making a commit any time I get compiling code, passing tests, or any other milestone - even a small one. THis IS how you can safely experiment with AI without fear of losing your work.

> I'm not saying push to the server or do a pull request (PR) every time - just a local commit is sufficient for this purpose.

Sometimes the AI will go off the rails and make a mess of your code. Having a recent commit allows you to quickly get back to a known good state.

## Create and Use MCP Servers

As you might imagine, I use CSLA .NET a lot. Because CSLA is open-source, the Copilot AI generally knows all about CSLA because open-source code is part of the training data. The problem is that the training data covers everything from CSLA 1.0 to the current version - so decades of changes. This means that when you ask Copilot to help you with CSLA code, it might give you code that is out of date.

I've created an [MCP server for CSLA](https://github.com/marimerllc/csla-mcp) that has information about CSLA 9 and 10. If you add this MCP server to your Copilot settings, and ask questions about CSLA, you will get answers that are specific to CSLA 9 and 10, rather than older versions. This is the sort of thing you can put into your `/.github/copilot-instructions.md` file to ensure that everyone on your team is using the same MCP servers.

The results of the AI when using an MCP server like this are _substantially_ better than without it. If you are using AI to help with a specific framework or library, consider creating an MCP server for that framework or library.

You can also build your own MCP server for your organization, project, or codebase. Such a server can provide code snippets, patterns, documentation, and other information specific to your context, which can greatly improve the quality of the AI's output.

I wrote a blog post about [building a simple MCP server](https://blog.lhotka.net/2025/10/02/A-Simple-CSLA-MCP-Server).

## Conclusion

AI is a useful tool for software development, but it is not magic. You have to learn how to use it effectively, and you have to be willing to review and refine its output. By thinking of the AI as a collaborator, providing it with context, and using MCP servers, you can get much better results.

As an industry, we need to move past the idea that AI isn't useful, and start discussing how to use it ethically and effectively. Only then can we fully understand the implications of this technology and make informed decisions about its use.
