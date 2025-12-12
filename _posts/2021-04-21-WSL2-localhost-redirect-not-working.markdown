---
layout: post
title: WSL2 localhost redirect not working
date: 2021-04-21T00:00:00.0000000-05:00
categories: []
tags: []
published: true
permalink: 
image: https://blog.lhotka.net/assets/2021-04-21-WSL2-localhost-redirect-not-working/tweet.png
---
I was very excited to see [preview 3 of .NET 6](https://devblogs.microsoft.com/dotnet/announcing-net-6-preview-3/) come out, especially with the hot reload enabled for Blazor.

Not wanting to tamper with my main Windows environment, I installed dotnet 6 into my WSL2 Ubuntu instance.

And immediately discovered that I was unable to access my Blazor server endpoint from Windows via localhost. This _should be automatic_ and require no effort!

I tweeted my issue and got lots of feedback on how you need to do a bunch of manual steps, including editing files in the project and more. But again, this _should be automatic_ and should "just work".

So I tried it on a different physical computer, and it worked great! Ah ha! My problem is narrowed to my workstation.

I wasn't entirely surprised, as I've tweaked my WSL2 on my workstation to enable systemd (which isn't supported by WSL2). So I installed a _new_ WSL2 Ubuntu instance, _but it failed also_.

Fortunately @craigloewen provided a tip about "fast startup", which I thought might be a WSL2 feature, but actually turns out to be a Windows 10 feature.

![](/assets/2021-04-21-WSL2-localhost-redirect-not-working/tweet.png)

Thankfully there's already a blog post with [instructions on how to disable Windows 10 Fast Startup](https://stephenreescarter.net/wsl2-network-issues-and-win-10-fast-start-up/) to resolve the WSL2 localhost redirect failure. Why this doesn't appear anywhere near the top of searches about being unable to access WSL 2 websites from localhost I don't know, because it did fix the issue.

In summary, if you create a web site, web app, or Blazor app in WSL2 and do `dotnet run`, you should be able to open a browser in the Windows host and navigate to localhost. It should just work.

If it doesn't work, apparently this fast start issue is the first thing to try. And if turning off fast start doesn't solve the problem, _then_ you can try all the workarounds like changing the targets to 0.0.0.0, editing host files, and all the other irrelevant workarounds I tried.