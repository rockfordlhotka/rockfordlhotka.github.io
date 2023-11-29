---
layout: post
title: How Does Blazor Compete with MVC and Razor Pages
date: 2023-11-29T00:00:00.0000000-06:00
categories: []
tags: []
published: true
permalink: 
featuredImageUrl: 
---
In a recent LinkedIn post, my friend and colleague [Rachel Appel](https://www.linkedin.com/in/rachelappel/) summarized a Jetbrains survey that included a question on which ASP.NET Core frameworks are being used.

![JetBrains Survey]({{ site.url }}/assets/2023-11-29-How-Does-Blazor-Compete-with-MVC-and-Razor-Pages/jb23surveyblazor.png)

Her question is this:

*JetBrains published our [DevEcosystem 2023 Survey](https://www.jetbrains.com/lp/devecosystem-2023/). Here is an interesting data point for ASP.NET Developers: Only 16% and 12% of ASP.NET Core devs are using Blazor Server and Blazor WASM, respectively. Why do you think this is?*

*Why are MVC and Razor Pages still more popular?*

There are many ways to answer. Let me try a few.

## Blazor Maturity

Blazor is just now on "version 3", which is the point at which most Microsoft technologies are considered stable and mature. 

Blazor 8 really is quite good overall, and has the basic components necessary to build enterprise apps. If you look at Blazor adoption, it has been growing, and I suspect Blazor 8 will boost those numbers.

> Personally I think the new automatic render mode feature, while magical, has some issues. It may be that .NET 9 is the better point to jump in (assuming they fix the issues). See this blog post about [managing user state in Blazor 8](https://blog.lhotka.net/2023/11/28/Per-User-Blazor-8-State) for one issue and possible solution.

## Applicable Scenarios

Blazor 6 and 7 were useful only for business app development, not to create web sites. Blazor 8 (with the server-rendered mode) is now also useful to creating web sites - so it now actually competes with Razor Pages, and _also_ continues to compete with SPA frameworks, while offering the ability to run all your code on the server *or* on the client (and now both automatically).

In other words, until .NET 8 Blazor was pretty much limited to building smart-client user experiences along the lines of Angular, Windows Forms, WPF, etc. The reach of Blazor 8 is _much_ broader, and may be attractive to a significantly wider audience of developers.

I think the increased scenarios is hard to understate. Blazor provides a _consistent programming model_ to create web sites, server-side web apps, and client-side web apps. All using C# and .NET (and probably SQL), which is a _far simpler_ tech stack than any of the "full stack" offerings can claim.

Also, I should point out that the Blazor MAUI hybrid scenario is quite compelling for building mobile apps. The same consistent Blazor programming model, wrapped in a native app for iOS, Android, Mac, and Windows, with full access to device resources and capabilities.

## Conference Attendance

At [VS Live](https://vslive.com) over the past 1+ years I've been giving a workshop on Blazor. It has consistently been the biggest draw, and in Orlando in November it drew over 3 times the number of attendees as any other workshop!

Over half the people in the workshop had not yet looked at Blazor. This was surprising to me, given that so many people have been coming to the workshop in previous events - but I guess it is like the early days of Angular, where there are seemingly unlimited numbers of people who want to learn the new technology.

This gives me a great deal of hope that the usage numbers for Blazor will continue to grow, and I suspect over a long period of time will ultimately eclipse MVC and Razor Pages.

## Existing Investments

I think it is very important to keep time in perspective. Enterprise development of web sites and apps takes a long time, is a massive investment, and enterprise apps are typically maintained for 10-20 years before being replaced.

A lot of organizations are still stuck maintaining Windows Forms and WPF apps, which are getting long in the tooth. These are, in my view, prime candidates for being rewritten in Blazor.

Most organizations have large investments in MVC and Razor Page style server-side web sites. Over time, these web sites will also become candidates for updating, and Blazor is (now) a compelling option to get a great mix of static web site plus interactive web app behaviors.

In both cases, smart client Windows and ASP.NET web sites, the developers already know and use .NET, so Blazor is a natural option.

Many organizations have invested in building single page applications (SPA) using Angular, Vue, React, or another framework. Some of these apps are quite new, others are quite old - even still in AngularJs! Those that need rewriting should, in my opinion, _consider_ the use of Blazor.

In the SPA case though, the developers might not be good at .NET, having invested so much in their web-based platform skills. Blazor may or may not be a valid choice here due to existing skill sets, yet I think it is something an organization should evaluate.

## Simplifying the Stack

From about 1993 through 2010, Microsoft developers enjoyed a ridiculous level of productivity. This is because "full stack" at the time was a singular technology to build back-end and client application code, plus SQL to talk to the database.

Around 2010 the industry decided that much productivity wasn't good, and we chose to build our client app in one tech stack, our back-end in a different tech stack, and we even chose to complicate our data storage with all sorts of NoSQL data stores.

> That was sarcasm. We didn't _choose_ this, it was thrust upon us with the rise of the iPad and the reality that it was no longer realistic to "just target Windows" for enterprise client apps.

Sarcasm aside, those of us who had enjoyed nearly 20 years of such productivity were dealt a serious blow, and even with the rise of nodejs the "new world order" has never come close to recapturing what we had.

Until now.

Blazor offers the same productivity and lower-cost development and maintenance for apps that we enjoyed for so long. We're back to where a dev team needs to know one tech stack (.NET) to build the client and back-end software, and then also needs to know one or more database technologies, still often SQL.

Those who enjoyed this kind of productivity are later in our careers now, and this is a breath of fresh air.

Those who have never experienced this kind of productivity are in for an awesome experience that is hard to describe until you've experienced it.

## Conclusion

Blazor is still quite young, and its usage has been steadily growing. It is only realistic to think that it will take years of continued growth to reach its full potential.

There are very real headwinds against _any_ new UI technology, including existing investments by organizations, developer skill sets, and platform maturity.

Will Blazor "take over the world"? I very much doubt it. I think we're at a point in history where there are a lot of competing ideas and technologies - which is good for innovation.

Will Blazor become a viable alternative to existing (and future) UI frameworks? I'm pretty confident the answer is yes.