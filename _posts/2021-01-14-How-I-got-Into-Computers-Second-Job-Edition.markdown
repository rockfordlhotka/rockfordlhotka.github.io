---
layout: post
title: How I got Into Computers Second Job Edition
date: 2021-01-14T00:00:00.0000000-06:00
categories: []
tags: []
permalink: 
image: 
---
This is the sixth post in a series about how I got into computers and how my career has unfolded.

* [Part 1 was pre-university](https://blog.lhotka.net/2020/08/15/How-I-Got-Into-Computers)
* [Part 2 was university](https://blog.lhotka.net/2020/08/17/How-I-Got-Into-Computers-University-Edition)
* [Part 3 was an internship](https://blog.lhotka.net/2020/12/27/How-I-Got-Into-Computers-University-Internship-Edition)
* [Part 4 was my first job search](https://blog.lhotka.net/2020/12/29/How-I-Got-Into-Computers-First-Job-Hunt-Edition)
* [Part 5 was my first job](https://blog.lhotka.net/2021/01/04/How-I-Got-Into-Computers-First-Job-Edition)

In my first job at ACS/Command Data I was part of the team that built the product on which the company was based. I helped build the software that was sold, installed, and supported by my employer.

My second job was in IT (Information Technology) in a bio-medical manufacturing company. In this job I was part of the team that supported the people who marketed, sold, manufactured, stored, and shipped the products for the company. The company's original name was INCSTAR, back in Minnesota where my wife and I were much closer to family and friends.

I held a number of different roles/jobs while at this employer, and I'll break them into different blog posts to avoid having a monster post covering six years of work.

INCSTAR was not a big company, perhaps 300 employees. The company did everything, from marketing to sales to sourcing materials to manufacturing to inventory to shipping to invoicing. End to end.

And the bio-medical products were, of course FDA regulated. They also were radioactive and so were regulated in that regard, plus standard manufacturing practices, and more. So not only did the company do all the "normal" manufacturing things, but the QA team (responsible for ensuring regulatory compliance) and QE team (responsible for product quality) were powerful and pervasive.

### Manufacturing vs Software Salaries

Being hired into a technical company, at the end of the 1980's, where the company's use of computing was extremely limited, as very educational. The HR department really had no idea how to structure the pay scales for people like my friend Tom and I as we joined the company.

Part of it was that HR was focused on pay scales for bio-chemical engineers, sales people, folks that working on the manufacturing floor, inventory, and shipping. The fact that software developers (as we're now called) _actually work in a parallel industry_ was beyond their understanding or experience.

As a result, we were (in retrospect) underpaid.

Also, Tom started at a higher pay level than I did, due to seniority. This, despite the fact that my _expertise_ was greater. Not to disparage my friend, as he's an amazing developer and has a set of skills and a worldview that is _very_ complimentary to my own. At the same time, I did (and still do) live and breathe software development and tech. That has often made me relatively unique, and this was true back then as well.

Still, we got hired, had jobs, and were able to move back to Minnesota to be near family and friends. And, if I look back on it, the pay jumps from my original hiring at ACS, through the Alabama move, and back to Minnesota, did get my salary to a level where my wife and I were able to (with care) have a workable living.

Living in AL for a time had allows us to accumulate some very modest savings, and that allowed us to get into what we thought was a decent apartment. We were wrong, it was a poorly run complex, and the apartment itself was nearly impossible to keep warm in a Minnesota winter. Looking back on it, the apartment was a horrible experience in many ways.

In the meantime, my wife got a job as a hostess for a restaurant within walking distance, because we could only afford one car, and I needed that to get to work every day, at a distance of about 10 miles each way.

I should point out that our titles at the time were something like Programmer I and Programmer II. In this and my other blog posts I tend to use _current_ jargon like "Developer" and "IT", but those terms weren't in vogue in 1989.

Also, my friend and I were hired shortly before a Director of IT was hired. So within a short few weeks, when we were starting to get just a sense how the company worked, Dan Dylla was hired as our boss. Spoiler alert: Dan was instrumental in building me into the person I am now, and I'll be forever grateful for the opportunity to work for him.

### Work Environment

Back to INCSTAR. The IT group (actually called IS: Information Services) occupied a little nasty nook in a back corner of the administrative building. The cubes were run down, and the computer room housed a VAX 11/780 and _a lot_ of heavy serial cables that ran to all the VT100 terminals around the building. Well, ok, _most_ of the old serial cables actually went nowhere, because as things had changed over the years they'd just be cut, with ends feeding into one wall of the computer room.

There was also a MicroVAX for development work, so we had an environment on which to work and test without risk of bringing down production. Despite the name, a MicroVAX was actually pretty powerful, though it was limited _via software_ to run slower. Unless you knew the workarounds to partially defeat the artificial limits.

The upside to being in a little back corner was that I brought in a "boom box" and we were able to listen to the radio without bothering other people. This was kind of nice, but over time I did discover that _us_ being able to listen to music caused envy from some folks in other parts of the company where that wasn't possible. We were viewed as being spoiled.

That's kind of a recurring theme come to think of it. Over time our little group started playing casual card games over lunch every day, and the fact that we actually took our hour-long lunch period and enjoyed ourselves also led to discontent among other parts of the company.

I always found that frustrating, because many of the most vocal people in that regard were smokers (tons of smokers existed back then!), and they would take numerous smoke breaks throughout the day, adding up to well over the one hour of lunchtime where we enjoyed playing Hearts.

I don't entirely know what conclusion to draw from this bit of pettiness. Maybe that people who have a good thing (numerous smoke breaks to visit and relax) have a hard time letting other people _also_ have a good thing when that other thing is different?

### Admin vs Developer Tension

The VAX system administrator was (it seemed to me at the time) an older guy who seemed to lack initiative. I, coming from an ISV where we developers had direct control over our development VAX, immediately started to take steps to improve at least the MicroVAX to be faster and more useful.

> He wasn't really old, just older than someone in their early 20's. Nonetheless, he had no sense of urgency about him, nor any real curiosity or drive to be on the leading edge of knowledge about tech.

Coming from a world where we built commercial software with the user experience in mind, I was rather taken aback by the fact that we just threw users to the command line without even a basic menuing system. So I wrote a menuing system, inspired by the one we provided to customers in my previous job. It took some work to convince the crusty VAX admin dude to run it in production, but it was immediately a feather in my cap across the company.

Ultimately his reluctance to improve, and probably his clear lack of admin knowledge when compared to _a developer_, led to him departing the company and me getting his job, which will be another blog post.

### Reusing Existing Knowledge

Since my friend Tom and I were the only developers at the time, we leaned heavily toward reuse of a lot of concepts from our previous job. Of course we were unable (legally, ethically, and technically) to bring code from our previous job to this one, but developers always carry their experience and knowledge with them everywhere we go.

> When I say we were technically unable to bring software, this is largely true. Minicomputers at the time generally used magnetic tapes for backups, and even if I had _wanted_ to unethically grab software, that would have meant doing a backup onto a magnetic tape reel to bring with me. I call this out, because _today_ that sort of technical barrier doesn't really exist: we all tend to work in environments that are cloud-enabled and carrying info means just copying it to a cloud location. Were one to ignore the legal and ethical concerns, there's no real _tech_ barrier.

What this meant, was that we had a set of coding practices and patterns that led us to be productive, but we had none of the underlying support software or tools.

I couldn't abide that lack of productivity, and so spent my "off hours" creating tools and frameworks to fill those gaps.

Dan (our new boss) explained a salaried job like this (as I remember it): you aren't paid by the hour, you are paid to get stuff done. Generally, salaried employees are expected to work 50-55 hours a week, and you should work on high priority things first, then high value things, then anything else.

The software we wrote for users was generally high _priority_, and sometimes was high value. Other times it was tweaking a report slightly or something like that. Creating developer productivity tools/frameworks was clearly (to me) high _value_ because the existence of these tools allowed us to finish high priority work faster.

It is also important to realize that we had no support organization. Any bugs, crashes, lost data, or whatever _always came back to us to fix_. When I was first hired there wasn't even a help desk; end users literally called us developers with their software problems.

This is a major driving force (in my mind) for creating software standards, tools, and frameworks. If every app is a one-off it rapidly becomes impossible to build, maintain, _and directly support_ very many apps. It is critical, in such a constrained environment, that software _just keep running_ without ongoing maintenance and support.

As a result, the tools and frameworks that we ended up with not only provided a consistent user experience and developer productivity, but they also provided high levels of maintainability and stability.

Over the six years I was at INCSTAR we built, maintained, and supported nearly 100 individual apps, _in addition_ to the primary ERP system used by the company. All that with a small team, and an IT budget that never exceeded 1.5% of corporate revenue.

Somewhere in a box in my house I think I still have the documentation I wrote for the INCSTAR UI, data access, and other frameworks. And the various developer tools, like a `make` command and all sorts of other things.

### FORTRAN and BASIC

The ERP system used by the company was called MANMAN. It was technically an _MRP_ system: manufacturing resource planning, not a full ERP system. It was written in VAX FORTRAN, which was one of the most amazing FORTRAN implementations ever created.

> Don't get me wrong, I'm not a big fan of FORTRAN, but the DEC FORTRAN compiler was famous for is optimization, feature set, and overall capabilities.

The system was built on top of a hierarchical database engine. The database engine included a preprocessor for FORTRAN, so as part of your software build process step 1 was to run the preprocessor to convert the database commands into FORTRAN.

This was a pain. A simple database command like `FETCH` could generate _pages_ of FORTRAN that I didn't write. Yet if I'd made a mistake in my code or in the `FETCH` statement I was left debugging all that FORTRAN that I didn't write!

Worse, we didn't have an interactive debugger like in the modern world today. Mostly, debugging involved examining the break point dump of the code and trying to infer what went wrong. And, of course, lots of `print` statements to print output to the command line. Super primitive by modern standards.

> Well, until _really_ modern cloud development, where printing to the console is extremely popular because you can't always attach a debugger to your container running in AWS or Kubernetes or whatever. That's part of why I'm writing this blog series actually, because "modern" software development has regressed to the way the world worked in 1990, and I think people should be aware of this sad reality.

My friend Tom and I rapidly decided that any software _we wrote_ would be in VAX BASIC and would minimize the use of these preprocessor libraries. We couldn't entirely avoid them, but we could minimize their use.

The reason we kept using VAX BASIC was, in part, familiarity. But also because VAX BASIC had all the _functionality_ of VAX FORTRAN (though not the amazing optimizing compiler), and a _whole lot of other productivity features_ that weren't available in FORTRAN. That's because FORTRAN was governed by standards, while VAX BASIC just took the best parts of FORTRAN, Pascal, and who knows what other languages. As a result, in my view, it was by far the most flexible and powerful language available for that platform at the time.

Still, a lot of our work was altering, enhancing, and customizing MANMAN. We had its source code, and we commonly changed that source code to meet user requirements.

### Our First MANMAN Upgrade

A few months into working at INCSTAR we needed to upgrade to a newer version of MANMAN. This was painful.

Before I get into the pain though, let me explain _why_ we had to upgrade.

Many industry analysts divide organizations into type "A", "B", and "C".

* Type A orgs are on the bleeding or leading edge of tech: they install the new OS in beta, or right when it is available
* Type B orgs install the new OS after it is clear that the version is stable, and usually after 1 or 2 patches have come out
* Type C orgs install a "new" OS because the one they are running is going off support

When I joined INCTSAR, the company was very much a Type C org, and the VAX operating system version was at end of support, forcing an upgrade. To upgrade the OS meant we also had to upgrade MANMAN, because the ancient version we were running couldn't run on the newer (but still very old) operating system version.

This was one of those crystalizing points in my career. The company had customized MANMAN quite a lot; before I was hired, and of course after I was hired.

The new version of MANMAN came to us as new source code. With tons of changes from the vendor that sold MANMAN of course, and we spent months figuring out what we'd changed in the old software, then reimplementing those changes into the _modified and updated_ new MANMAN code. It was tedious, painful, slow, and expensive.

Perhaps worst of all, the users were increasingly frustrated because we had so little bandwidth to do anything _other_ than this upgrade, so we were meeting very few of their requests for ongoing enhancements and features.

When we finally did complete the MANMAN upgrade we had a really long list of user requests to address. These days that's called a backlog, and it was extensive.

The lesson I took from this, after lots of conversations with Dan and others, was that we would minimize any modification to MANMAN going forward, and would work to replace existing modifications with custom apps over time. None of us wanted to go through such a painful upgrade process every again.

Personally this made me very happy, as it was a lot more enjoyable to write and maintain software _outside_ MANMAN than by modifying MANMAN. And by this point we were starting to accumulate a pretty decent set of tools and frameworks for building custom apps, all of which made life generally better for us, and made us more responsive to user requests.

This was also the time our crusty VAX admin left the company. I wasn't privy to whether he left voluntarily or was fired, but in any case, he was gone, and somebody needed to assume the admin mantle. That ended up being me, which is a topic for another post.
