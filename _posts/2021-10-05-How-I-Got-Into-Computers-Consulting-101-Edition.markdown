---
layout: post
title: How I Got Into Computers Consulting 101 Edition
date: 2021-10-05T00:00:00.0000000-05:00
categories: []
tags: []
published: true
permalink: 
image: 
---
This is another in a series about how I got into computers and how my career has unfolded.

* [Part 1 was pre-university](https://blog.lhotka.net/2020/08/15/How-I-Got-Into-Computers)
* [Part 2 was university](https://blog.lhotka.net/2020/08/17/How-I-Got-Into-Computers-University-Edition)
* [Part 3 was an internship](https://blog.lhotka.net/2020/12/27/How-I-Got-Into-Computers-University-Internship-Edition)
* [Part 4 was my first job search](https://blog.lhotka.net/2020/12/29/How-I-Got-Into-Computers-First-Job-Hunt-Edition)
* [Part 5 was my first programming job](https://blog.lhotka.net/2021/01/04/How-I-Got-Into-Computers-First-Job-Edition)
* [Part 6 was my second programming job](https://blog.lhotka.net/2021/01/14/How-I-got-Into-Computers-Second-Job-Edition)
* [Part 7 was my foray into IT Pro work](https://blog.lhotka.net/2021/03/29/How-I-Got-Into-Computers-IT-Pro-Edition)
* [Part 8 was my return to app dev](https://blog.lhotka.net/2021/04/05/How-I-Got-Into-Computers-AppDev-Edition)
* [Part 9 was about my final job search (I hope)](https://blog.lhotka.net/2021/05/05/How-I-Got-Into-Computers-Final-Job-Search-Edition)

In this post I'll discuss my first role in the world of consulting, working for a company named BORN.

BORN was named after its founder: Rick Born. I don't remember my exact employee number, but I think it was around 160, with Rick being employee number 1. So I wasn't in on the ground floor, but was an early hire. The company was small and rapidly growing.

Some consulting orgs are "head hunters", either employing their folks as 1099 contractors, or as W2 full-time employees, or in other financial models. Regardless, these firms typically only pay their employees when they are billing.

Others, like BORN, only employ W2 full-time employees, and they pay their employees the same whether the consultant is billing or not. Obviously this means the company is _highly_ incented to keep consultants billing, because when a consultant sits idle ("on the bench") the company loses money.

I liked the employment model because it provided a stable income with good benefits. Being a self-employed consultant, and in some cases head hunter consultancies, can result in much higher incomes, but with much lower stability. As a relatively new father, stability was important to me.

My first consulting engagement (gig) was with a fairly well-known company that built components for hard drives. Unfortunately, the company was roughly 90 minutes driving through the city and into the rural area to the west of the city. And that was with no traffic, which obviously was never the case. So in practice my commute was well over 2 hours each way, so consumed at least 4 hours of each day.

That was a brutal start to my consulting career, to say the least!

Fortunately, the client had a small satellite office on the west side of the Twin Cities. With traffic _that_ was a 60 minute drive each way. I was able to work in that office 2-3 days each week. The drawback to that office is that it was almost always entirely empty, leaving me working in a very isolated location.

The work itself was to write some data collection software in Visual Basic 3, with the wrinkle that I was developing and deploying to OS/2, not Windows. I know, OS/2 had its proponents, but it was a very slow and clunky experience compared to Windows at the time. Yes, it was probably better architecturally in many ways, but its usability was quite poor.

What I remember most about working with OS/2 was the contrast between IBM documentation for developers compared to the docs from Digital Equipment (DEC) for the VAX. It wasn't really a fair comparison, because DEC was famous for its fantastic docs, and IBM was not. It was painfully (to me) apparently why IBM had no reputation for docs: they were nearly useless, because they assumed so much pre-existing knowledge of the IBM ecosystem that they were nearly impenetrable.

Later, when I became an author, I learned that what they lacked was called "sign posting", or providing clues to the reader that provide context and references to the meaning of jargon and background concepts.

There were four things I found valuable from working with the client.

First, I got to work with another developer (their employee) named Scott. Scott had a disability where his muscles weren't entirely under his control. He had a plastic shield over his keyboard, with finger-sized holes over each key. This allowed him to basically toss his arm onto the keyboard, then slide his hand into position so he could put a finger through a hole to press the desired key.

Although Scott wasn't able to type rapidly, he was a smart and dedicated developer, and his love for software development shone through. He understood the problem domain, and the two of us collaborated to figure out how to get VB working on OS/2.

Second, my immediate supervisor was a good boss, but pushed me outside my comfort zone. At this point in my career I was very good with VB, and I finished building the desired software much, much faster than anticipated - even with the OS/2 complications. But my supervisor said I wasn't done, and that I needed to implement a rigorous testing suite for the software.

I'd never really encountered the idea of quality engineering until that point, and I'm not sure that was the term yet. But that's what he was after: take the viewpoint of trying to prove the software would fail, not the typical developer worldview of proving that the software worked.

Not that I _enjoyed_ that type of testing. Nor was I good at it - probably because I didn't (and still don't) think it is fun. The thing is, as my career has progressed from that point, I've come to appreciate just how critical QE is to success, and I really appreciate people who _do_ enjoy testing software to try and break it.

Third, this was my first exposure to source control. The tool was PVCS, and it was horrible. Checking out or in the code would often take 20 minutes - for a tiny little bit of code. It took a while, but eventually I understood two things: that 20 minutes of idle time _for me_ was still billable work time, and that source control was (if it were faster) far better than nightly backups for tracking code changes.

Up to this point in my development experience, the only "code history" was maintained through change comments in the code, and nightly backups of the hard drive to magnetic tape. Although checking in/out code was painfully slow, or impossible when PVCS went down (frequently), the potential benefits were pretty obvious.

Over time I understood that me being idle for chunks of each day wouldn't be penalized by the client (because they suffered too, and accepted the consequences of their tooling). And I realized this "idle time" surely wasn't penalized by my employer because a billable hour is a billable hour whether it was productive _to me_ or not. BORN's two measures of success were that the client was happy (they were), and that my time was billable (it was).

Still, I _personally_ found the idle time extremely frustrating. I'd never had a job where non-productive time wasn't frowned on.

Fourth, other developers (employees) at the client were digging deep into some of the early (this was 1995) ideas around mainstreaming object-oriented programming, applying what would later become agile processes, and other concepts that were new to me, and that were extremely thought provoking.

I didn't get to apply those concepts at the time, but it was great to be exposed to them, and to listen to more experienced developers discussing the books and ideas.

Between the 2-4 hours per day commuting, the frustrations of working alone in an office, and all the idle time that came from PVCS, I can't say that I really enjoyed the project. But the client thought I did a good job, because once the software was tested and signed off, they kept me as a billable consultant _while they waited for another project to come along_!

So I drove 4 hours each day to sit in their office and - do nothing.

I raised this issue to my boss at BORN. He didn't see the problem, as I was billing and what else mattered? I was pretty insistent though, because being paid to do nothing, coupled with a daily 4 hour commute - well that wasn't tolerable to me.

Fortunately, another VB gig came up and they switched me to a different client, and that'll be another blog post.

Before I let you go, I want to contrast the jobs I had to this point.

I started out working at an ISV, where _I was building the product being sold_.

Then I worked in IT, where _I supported the folks building the product_.

This consulting work is different yet, because _I was the product being sold_.

Different folks are drawn to the pros and cons of each type of job, and although I've been in the consulting _field_ since 1995, I can tell you that, for me, it is my least favorite type of job.

I get a lot of satisfaction in watching software deployed, seeing people actually use the software, and in seeing the real-world impact of my labors.

Consultants (in my experience) almost always move on to another client or project when the software is done, and before it is deployed. As a consultant I never once got to see the software actually used, nor did I get to see the real-world impact of what I'd created.

Yeah, I got paid. I've built some very cool software. Yet it has never been satisfying _for me_ because I have no idea if it was well received, or changed people's lives, or enhanced the company. 

I just wrote a blog post about how I think most [non-software companies should prefer consultants over hiring software developers directly](https://blog.lhotka.net/2021/09/13/Developers-Hire-or-Use-Consultants). That wasn't an argument _for me_, but objectively from a business perspective.

Personally, my preference would be to work for an ISV, building the actual product being sold. Taking customer feedback and using it to enhance the product over time. _That_ is satisfying to me!