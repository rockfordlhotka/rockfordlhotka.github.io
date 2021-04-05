---
layout: post
title: How I Got Into Computers AppDev Edition
date: 2021-04-05T00:00:00.0000000-05:00
categories: []
tags: []
published: true
permalink: 
featuredImageUrl: 
---
This is another in a series about how I got into computers and how my career has unfolded.

* [Part 1 was pre-university](https://blog.lhotka.net/2020/08/15/How-I-Got-Into-Computers)
* [Part 2 was university](https://blog.lhotka.net/2020/08/17/How-I-Got-Into-Computers-University-Edition)
* [Part 3 was an internship](https://blog.lhotka.net/2020/12/27/How-I-Got-Into-Computers-University-Internship-Edition)
* [Part 4 was my first job search](https://blog.lhotka.net/2020/12/29/How-I-Got-Into-Computers-First-Job-Hunt-Edition)
* [Part 5 was my first programming job](https://blog.lhotka.net/2021/01/04/How-I-Got-Into-Computers-First-Job-Edition)
* [Part 6 was my second programming job](https://blog.lhotka.net/2021/01/14/How-I-got-Into-Computers-Second-Job-Edition)
* [Part 7 was my foray into IT Pro work](https://blog.lhotka.net/2021/03/29/How-I-Got-Into-Computers-IT-Pro-Edition)

In this post I'll discuss my third role at my second job. My first role at INCSTAR was as a programmer working in IT and my second was supervising the IT help desk, PC support, and system admin group. My third role was to lead our team of programmers.

The guy who'd been leading the programming group left the company, and I had been doing the IT Pro thing long enough to know it wasn't for me. Fortunately Dan, my boss, was willing to shift me to leading the programming group and he hired someone new to run IT operations.

## Handling Mergers

As I mentioned in previous posts, our company kept growing: organically and through acquisitions and mergers. By this point we were owned by a parent company and had absorbed a comparably sized org from Cambridge, MA into our Minnesota location. I probably forgot to talk about that in my previous post, though in the end it wasn't a terribly big deal for us in IT, as they switched to using our existing systems and so it was mostly provisioning new users.

But as the head of programming it became a big deal, because the Chief Operating Officer (COO) from that other company was aghast at the lack of metrics and reporting in our environment. He brought a lot of maturity around manufacturing that I never knew we were missing, and this was good for me, because I had to understand what he wanted/needed in order to deliver it.

## Seeing The Bigger Picture

Dan, my boss, had a regular refrain with me: step back and see the bigger picture. I was a techie, and would jump to solutions very rapidly. I was frequently right, but often not right in the best way, because I didn't take time to understand the _big_ business issues at play. Nor did most users asking for help from our group.

Mostly users wanted (and this has been true my entire career) small, tactical solutions to their immediate problems. Very few users are big picture thinkers, and very few look outside their little work area or niche to see how their problems come from elsewhere, and how their solutions might cause problems to flow downstream.

Dan's constant refrain eventually did sink into my worldview. And I give a great deal of credit to Dan for my subsequent successes, because he was absolutely correct. _Someone_ needs to step back and look at the big picture. Most users don't and won't, leaving it to us application developers to do that heavy lifting.

_Today_, decades later, we often address this through "agile teams" or similar schemes. And this is better than it was back in the early 1990's for sure! Still, working in consulting, I get to see a lot of different orgs and teams, and it is still true that most people aren't "systems thinkers". Most people aren't particularly good at taking a problem, stepping back to see external relationships to that problem. Nor are most people good at then breaking that _bigger_ problem space down to its essence to identify critical paths, unintended consequences of the existing and potential future state, and then assembling a plan to change the existing problem space into something new. Sometimes doing all this even requires software!

I probably sound arrogant. That's not my intent. In fact, my intent is to suggest that this skill set isn't exactly _rare_, but it is uncommon. Most people don't have it. Sadly, that sometimes includes folks who work in the IT and software industries. Fortunately this is just a skill set, and so _people can learn it_ and work to improve their skills and abilities.

I know this to be true, because prior to Dan's coaching I never bothered to step back and understand the big picture. Or to understand that, in so many cases, software is a mere component in a much larger business process change.

For example, while managing the programming team, we identified the need to have a computerized document management system. We explored some products that were just out at the time, but they were extremely expensive and didn't come close to meeting our needs. So over about 18 months I created the software component of a new document management system.

You might think that a document management system _is just the software_, but it involved changing part of the building to have a document vault with its own fire suppression and security, major changes to document authoring and review processes, gaining approval for "digital signatures", substantial changes to how work orders were placed for manufacturing, how they were tracked through manufacturing, and processed and filed at the end.

So yes, the software was a big deal, I spent over a year building it. But I also recognize that the software was just one piece of a big puzzle, and it was far from the largest piece.

Why was this so complex? Do you remember chemistry class? Maybe in high school, and for sure in university, you'd have done labs. I remember doing those labs, following worksheets where you filled out results, did some math, got more results, and ultimately got an answer. I always thought those worksheets were _for school_ and real chemists didn't use sheets like that. How wrong I was!

In bio-medical manufacturing, it is all chemistry and biology. And every work order to make a product meant printing out a packet of worksheets (and other documents) necessary to produce the product. Basically just advanced versions of the worksheets students fill out in chemistry labs in university.

This document management system I wrote pulled data from the MRP (materials resource planning) system, Word doc files (this all predates docx format), and merged them together to create the custom document packet necessary for each specific work order. To make this all work, we automated printing the Word docs to PostScript files, and then my software would alter the PostScript to merge in the data from the MRP system, plus other regulatory data and information from other sources. The PostScript file would then be sent to Apple laser printers (the best PostScript printers available at the time) for printing.

Kind of a Rube Goldberg sort of thing, but it worked very well. Building a packet by hand took hours or days, and now they would be generated during nightly processing, ready for the manufacturing floor every morning.

## Software and Ethics

Which brings to me to an existential discussion point. Before this document management system, we had 12 people working full time to create these work order packets. They'd pull original docs from file cabinets, photocopy them, transcribe all the MRP and regulatory data into the new documents, assemble all the worksheets and other docs into the right order, and create each packet.

My software automated away 11.5 of those jobs. The half-job remaining was that _someone_ still needed to pull the paper off the laser printers and put each packet in a binder.

One one hand you can argue that this document management system freed up 11 people to do more fulfilling and rewarding jobs somewhere in the world. None at INCSTAR, but hopefully somewhere. And we _immediately_ saved the hourly wages and benefits for 11 people, reclaimed a whole bunch of floor space where they used to do all that work, and radically reduced the error rate by changing a manual system into an automated system. Plus all sorts of security benefits and more.

On the other hand you can argue that we eliminated 11 jobs held by people at the low end of skill requirements. Jobs that required a high school education and nothing more, and so perhaps we reduced the overall employability of these individuals. _Certainly_ Dan got a reputation for getting rid of people, and I was known as "Dan's hatchet man".

> It didn't help that this was just the first of a series of groups Dan set his sights on, in terms of radical transformation of business processes, and each time I was there building the software to enable the transformation.

Here I sit, roughly 27 years later, and I don't have a real answer. The business value of automating repetitive work (office work, manufacturing via robots, etc.) is crystal clear. Capitalism dictates that companies _must_ engage in this sort of optimization to remain competitive, because _some_ company will automate away jobs with cheaper software/hardware.

At the same time, without a social structure to deal with "unemployables" who's jobs are steadily automated away, it is hard to see a bright future that includes everyone. This is particularly harsh in the US, where most people's personal identity and worth are so closely tied to their job. How good can you feel if your job can be offshored or automated, such that it is done for a tiny, tiny fraction of what it costs to eek out a basic living.

Now you get a glimpse into my head, dear reader 😉. It would be easy enough to gloss over these thoughts. To just enjoy writing software and not think of any consequences, but that's not how I was raised or how my mind works.

OK, without a firm answer, let me move on to another broad conceptual topic.

## Rewards of App Dev in IT

Writing software for a company, when you work for that company, is something I found extremely rewarding. My team and I wrote over 80 apps that offered added features beyond our core EPR/MPR system, accounting system, etc. This was quite good for a small team of (at our peak) 5 developers. Especially given that we were also _supporting_ all the software we wrote.

This was only possible because I (taking a page from my first job) insisted on strict coding standards, automatic management of things like log files and other maintenance tasks, and other things that made most of our software somewhat self-healing, easy for the help desk to troubleshoot, and often for the help desk to resolve.

Not that we were perfect! But when an "opportunity" came in via our Opportunity Tracking System to fix a problem with the software, our philosophy was to not just fix the problem, but to try and prevent it in the future, or to build alternative fixes that didn't require programming in the future. That philosophy is, in my view, why we were able to keep building new software, rather than being buried in maintaining the software we'd already created.

As a consultant, I've worked with companies that have a small group of "new development" folks, and hundreds (or thousands) of developers doing maintenance on existing systems. To me, this indicates a fundamental failure within the org, demonstrating a lack of balance between creating new software and creating _maintainable_ software.

On the other hand, those orgs _do employ_ hundreds or thousands of developers who don't have to learn new tech or concepts often, if ever. And perhaps many of those people are happy? I know I wouldn't be - that sounds like an ongoing nightmare from my perspective.

> I'm not kidding about not learning new stuff often. Most actual enterprise software systems go into production and last for 10-20 years. When an org spends millions of dollars building some software, it takes a _very long time_ to recoup that cost! This is why most big airlines and banks still run on COBOL, and why Windows Forms remains perhaps the most widely used UI technology in the Microsoft world.

Anyway, back to my point, which is that I find it extremely rewarding to write software _and watch it deployed and running_. To actually see users become more productive and hopefully happy because of my software.

This was something I rarely got from Command Data, because we'd write the software and it would be deployed by technicians at customer sites. Only a couple times did I actually get to go on-site at a ready-mix concrete or aggregate customer to see the software in use.

But working at INCSTAR, literally every day my team and I got to see people using our software. When it worked well we got smiles, or got ignored. When it didn't work well, we heard about it, and put that feedback into the Opportunity Tracking System (these days we'd put it on the backlog - same concept).

## Looking to the Future

One thing about working in IT is that things tend to move slowly. Although we were building new software on a regular basis, our _tool set_ changed very slowly. There's no business value in adopting new tools or programming languages just for fun. There needs to be a compelling reason.

Dan had me research the use of C++ on the VAX, because the trade magazines were making a big deal about it at the time. And it appealed to me personally, because I really enjoyed C, and this was the next best thing - at least in theory.

In practice, it turns out that C++ (like C) is built around some core operating system assumptions that reduce the value of the language outside the *nix world. This is particularly true on the VAX, where the file system was rich and powerful, not just a bunch of simple streams. Even worse, the VAX system services (what today is called an API) was created with FORTRAN and assembly language calling semantics that were (at the time) very challenging to work given the name munging that comes with C++.

That name munging issue has never gone away with C++, but abstractions have evolved over time, making C++ work on things like Windows and other environments. Back when I was evaluating the language on the VAX, those abstractions (to my knowledge) weren't available, or at least not mainstream.

In summary, things that took just a handful of lines of VAX Basic or FORTRAN took pages of C++, just to deal with the VAX system services and the fact that the VAX file system (RMS) wasn't just a store for streams, but did so much more.

To their credit, DEC did enhance the VAX file system to support simple stream files. And that was good for the C/C++ world, though using that file type meant you _weren't_ using all the awesome power of RMS.

Most of you reading this are probably scratching your head now, because neither *nix nor Windows has anything remotely comparable to the VAX file system. Those of you in the Microsoft space long enough might remember Windows Server (back in 2005?) that was going to have a super-advanced file system, which never materialized? I think that file system might have been vaguely similar to RMS.

But I digress. We evaluated C++, and as much as I personally _wanted_ to use it, the language introduced so much complexity compared to our existing toolset there was no way to justify its use.  
  
However, in 1991 Microsoft released Visual Basic 1.0. So Dan got me a "laptop" so I could spend some serious time learning this new thing. Along with some IBM competitor at the time (the name of which is lost in my memory). This laptop _was_ an actual laptop, it wasn't one of those luggables. By today's standards though, it was big, heavy, and the battery would last maybe 30 minutes or something.

I was also concerned about us not being able to adopt C++, and what this might mean to my career. After 7+ years of VAX Basic (which I truly loved), I was really ready for change!

So I saved and scrimped over a few months so I could buy a computer big enough to run this new Windows NT thing. I was attracted to Windows NT because the operating system was created by David Cutler, the same guy who created the VAX OpenVMS operating system. Better still, it ran Windows apps, plus had a POSIX subsystem to run *nix software.

And I bought a C++ compiler. From Symantic? It was expensive, with a super-complex IDE. Frankly, it was a huge mistake on my part, because I spent more time fighting the stupid IDE than learning how to build software with C++ on Windows.

> And what I didn't know until later, is that C++ on Windows suffered from many of the same productivity issues as C++ on the VAX. Simple "hello world" apps took pages of software just to deal with Windows APIs.

Basically, I put myself in a spot where I was comparing early Windows C++ to early Windows Visual Basic. And it was no contest in the end - VB was _so much more productive_ that C++ was laughable.

And yes, I know, I just insulted a lot of folks who did build create software on Windows with C++. I get it. But VB let me write software far, far, far more efficiently, and it turns out I value that aspect of development quite highly.

## Writing an Email App

In my last blog post I talked about setting up UseNet on the VAX via the uucp freeware package. That allowed the use of email in and out of the company via the same dial-up connection to my other friendly VAX users in the Twin Cities. But email on a VT terminal is pretty lame compared to having a GUI experience right?

So I set out to learn VB by building a GUI email frontend to the email system running on the VAX. Which isn't that tall of an order at a basic level, considering that the VAX software was just simple SMTP using a file system based inbound and output queue system. Pretty much like all SMTP systems of the time.

However, scope creep is a real thing! Pretty soon my iMail program allowed emailing between people _inside_ the company, using the Novell file system so people could email each other, send attachments, and so forth. And if they _did_ email an address outside the company, then all that email/attachment stuff would get routed to the appropriate SMTP queue directories on the VAX with proper encoding.

Perhaps not surprisingly, the entire company rapidly adopted and became reliant on email for communication, all based on my iMail app written (ultimately) in Visual Basic 2.0.

The most challenging part of this was that we needed to get email support to a remote site in rural Maine. They connected to our main systems via modem, and so I built a PC-PC gateway based on the Kermit data transfer protocol so emails could flow to/from this remote site. That was quite challenging, because the connection was slow and unreliable, so my gateway had to deal with retries, partial transfers and all sorts of edge cases.

Another challenge I remember is that I relied on the Novell file system as a shared lock manager. The VAX had a distributed lock manager built in, so if you needed a cross-process or even cross-computer lock for synchronization it was easy. Windows had (and I think still has) no such thing, but the Novell file system allowed locking files.

Sadly, Novell's file locks weren't _entirely_ reliable. Every few weeks the lock file would just stay locked, and the only resolution was to reboot the server. I don't recall whether we ever atually solved that issue, or if we just kept working around it. I do remember testing it on my Windows NT server and have no issues, but I never was able to convince Dan to replace the Novell server with a Windows NT server.

## Conclusion

INCSTAR continued to be acquired, including purchases where we were part of some bundle of companies. I think we were owned by American Standard Plumbing, and Fiat (an Italian car maker), and then some holding company. It was Fiat that was the beginning of the end for me, as they placed zero value on computers or automation. They just didn't understand why we had so many PCs, why we needed so many developers, and why the VAX wasn't good enough with its existing ERP system.

I could see and feel that software development was not valued by the new owners, and I found that quite depressing. Not least, because Dan and I and the whole team had done some truly amazing work over the years. I still look back at those years with a lot of pride of ownership, in terms of the people I worked with, the folks who's careers I helped get started, the software we built, and how we built it, and the amazing value we provided to the company.

Nonetheless, if the folks who own your org don't see the value in what you do, well, it is time to leave.

The next post in this series will cover my first entry into consulting.