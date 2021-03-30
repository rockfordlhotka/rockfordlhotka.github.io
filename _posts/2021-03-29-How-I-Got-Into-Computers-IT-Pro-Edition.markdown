---
layout: post
title: How I Got Into Computers IT Pro Edition
date: 2021-03-29T00:00:00.0000000-05:00
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
* [Part 6 was my second programming job](https://blog.lhotka.net/2021/01/04/How-I-Got-Into-Computers-Second-Job-Edition)

In this post I'll discuss my second role at my second job. My first role at INCSTAR was as a programmer working in IT. The opportunity for my second role came maybe 18 months later when our VAX system administrator left and we needed someone to run the IT/operations aspects of the company.

I was maybe 23 or 24 years old at this time, and was all about climbing the corporate ladder, so the opportunity to not only become the VAX system administrator, but also supervisor over the (even younger) guy we had doing PC maintenance seemed to make sense.

In retrospect, I'd probably make the same choice, though maybe not.

Pros:

* I did get a promotion
* I did get experience supervising and hiring people
* I learned the importance of logging/monitoring and self-healing software
* I was able to leverage a whole lot of deep VAX expertise I'd gained thus far in my career
* I learned all about early PC networking as we rolled out Novell Netware servers
* I learned all about supervising a help desk as we built out that function in the company

Cons:

* I like programming, and applied my skills to IT ops (DevOps anyone?), but it wasn't the right fit
* Running a help desk takes a temperament I apparently don't have
* _All the IT jokes are real_ - people really did hang floppy disks on their PC with a magnet, and everything else you've ever heard!!

So yeah, I'd do it again for the lessons I learned. But I'd never choose IT Ops or help desk manager as a job otherwise, as it just isn't for me.

To start with, I needed to step into the role of VAX system administrator. Which was fine by me, because I'd never thought the previous person did a great job. I spent a bunch of time tweaking performance, improving security, and optimizing various things like backups and how we managed printers.

Speaking of printers, this was a key part of being the system admin, because every night we printed boxes of white-green bar printer through these big floor-mounted dot matrix printers. Sales reports, operational reports, and many other things. Keeping the printers working, ensuring we had paper, and dealing with printer jams was challenging.

This led to my first hire: a part time night operator to help keep the printers working at night, make sure things actually printed, got sorted, etc. This was great, because this person could (and did) also make sure the backups ran correctly, and that all the nightly processing jobs on the VAX completed (which they often did not - without some help anyway).

I also inherited an employee who maintained the few PCs we had around the company at that time. Mostly 286 IBM knock-offs, that were very unreliable and frequently needed to be vacuumed out because they'd fail due to heat issues, failed parts, etc.

This person was also my first problem employee, because he generally did the least work possible and the most socializing possible. Very often that meant he'd "fix" a PC, only to have it fail a day or two later. The solution, ultimately, was to track every PC failure and repair, providing measurement and metrics of his work, and eventually of the quality of PCs we got from various vendors.

If you measure something, you almost always automatically gain substantial improvement in the thing you measure. My experience is this base-line improvement is around 10%, effectively for free, just by adding metrics and objective measures.

For my part, I also had the pleasure of writing an "opportunity tracking" system that we used to file job tickets, how they progressed, and how they ended. I originally planned it as "issue" or "problem" tracking, but my boss/mentor/coach Dan insisted that every problem was just an opportunity, so we called it the opportunity tracking system.

And you know, corny as it sounds, Dan was right. That term became part of our group's culture. Sure, we laughed about it, made some cynical jokes, but over time it truly changed the way people thought about "problems".

INCSTAR was growing over this time, and we now also had a way of tracking all the work being done by IT for users. Dan used that data to lobby for getting an actual help desk person. Someone to talk users through common computer issues, and otherwise enter opportunities into the system for our PC repair person, the developers, or me to address.

This was my second hire, and my first time working with a woman. Yes, seriously, up to this point all my immediate co-workers had been guys. Sure, Command Data had a couple women in HR and sales, and INSTAR had a lot of women working in sales, admin roles, and a fair number of scientists (it was a bio-medical manufacturing company). So, reflecting on the past, I clearly had _interacted_ with quite a few women in the workplace, and the scientists were highly accomplished and well regarded in the company. But still, the woman I hired was the first I'd worked with in any IT capacity.

She was a great hire, and really ramped up fast in terms of understanding user questions and challenges, and how to talk people though a lot of them. Not overnight over course, this took months. The thing was, unlike the PC repair guy who'd slip away from work given any chance, my help desk person and night operator person were reliable and fun to work with.

Eventually we hired a second help desk person, and he was also great to work with.

The night operations though, that was a constraint pain. Especially at week end, month end, and (shudder) year end. Most nights, all the jobs were likely to complete, or needed just a tiny nudge. The week end jobs would run through Friday night and into Saturday, and were less likely to succeed. The month end jobs would run Friday night though the day Saturday and often failed. And year end... Well, I'd just camp out to get through the roughly three days of processing that took.

This got somewhat better as we upgraded our modems, so it was possible to dial into the VAX from home. By this point I had an Amiga, with a decent terminal emulator, and with the new 2400 baud modems we were flying! When something failed that the night operator couldn't fix, I was able to dial in and usually fix it.

I was convinced I could make this better through software. I could automate the process, detecting failure, and switching from running one job at a time to numerous jobs at a time when appropriate (some jobs required exclusive database access, others didn't). Dan thought it was too complex and that I'd be wasting my time.

Have I mentioned that I have a temper? Most people don't know, because I'm generally very even keeled and chill, but I do have a temper. And one thing that can really get me going is when I'm told I can't solve something that I know I can solve.

So I spent many evenings and weekends dialing into the VAX, writing the software to manage all the nightly processing. It was a problem space I knew intimately, because when it failed I had to fix it.

The OpenVMS operating system on the VAX had a fantastic batch processing system, where you could queue up jobs and have them run singly, or in parallel, or even across multiple machines on the cluster (if you had a cluster). INCSTAR didn't have a cluster, but did have a multi-CPU computer, so parallel was valuable for jobs that didn't need exclusive database access. Still, some jobs required a lot memory, or IO, and others didn't.

So my management software was built to understand the needs of the various jobs - memory, CPU, IO, exclusive database access, and dependencies between jobs. It submitted jobs into the VMS batch queues as appropriate, changing the queues from single-job to 2 jobs, to n jobs depending on what would max out the use of the computer to get the work done fastest.

Even better, it understood failure conditions and dependencies. So failure of one job wouldn't stop _everything_, only jobs that depended on that one job.

In short, it made the whole night operations problem almost infinitely better, because everything almost always succeeded every night, even quarter and year end. Not only that, I cut all the run times roughly in half, due to optimizing the use of resources. This made quarter and year end _way better_ for accounting and other parts of INCSTAR - people who always had to wait for these jobs to complete in order to do _their_ quarter and year end work.

Was it worth the time I "donated" to the company? I think so, for multiple reasons.

1. I worked out my anger with Dan
2. I like sleeping, and not being regularly woken up to fix nightly processing was a win
3. This was cool software to write! Very challenging, complex, and rewarding to create
5. Now, decades later, I can honestly say I was a "devops pioneer" 😉

Did I get recognition? I think so, though this was so long ago I'm not entirely sure. I remember having to sell Dan on it, using it to run Friday night processing to establish that it worked, and worked better than what we had before. I think _he_ thought I was crazy for having done all that work on my own time.

Oh, I almost forgot. Along in here, we moved our data center. INCSTAR had built out a whole new section of building to house offices, freeing up the old building for more manufacturing and R&D space for labs. The new building had a very large data center, and I was responsible for getting the VAX moved and running. It was nerve-wracking to be pushing such a super-expensive computer, the size of a large refrigerator, down hallways and elevators to get to the new data center!

Two interesting stories about this. One is that the data center room was designed with the assumption that the company would switch from VAX minicomputers to IBM mainframes. That didn't happen (thankfully!), but this led to a problem. The HVAC system in the room was designed for IBM mainframes and the heat they generate. Our VAX computers didn't generate enough heat, so the HVAC system kept freezing up. In the end, half the system had to be run on heat, and the other half on cold, so the result was that the computer room was maintained at the right temperature without freezing the HVAC system.

The other "interesting" story is that the room was outfitted with a halogen fire suppression system. There was a big red button just inside the secure door that you could press to prevent it from going off. Say, for example, that the system detected a false alarm, you had 60 seconds to push the big red button, or the system would go off.

What we hadn't thought through, was that the electronic access pads would also go offline. So the alarms go off, we run to the door _and can't get in_!! Dan runs back to his office to get the physical key, but it was too late. It was kind of fun to watch the room fill with white stuff, and then eventually clear.

It wasn't so fun to clean up after, and it turns out that recharging these systems is kind of expensive.

That led to another team exercise: creating a disaster plan that included things like fire, false alarm fire, etc. Lack of disaster planning is just a disaster waiting to happen!

Another big thing I did while working in IT Ops was to install an early Novell Netware server. The use of PCs in the company was expanding rapidly, and we needed a way for users to share files, and also to more easily get files to/from the VAX.

So we bought a "big" 386 server and I tediously installed Novell on it. I don't remember how many 3.5" floppies, but it was a lot of them. And the install didn't go right the first couple times, so it took days. 

Then we had to plan out running network cabling through the company. This thing called "ethernet" was noise in the background at the time, very experimental. We went with twisted pair ArcNet. A lot of people used ArcNet via coax cable, but twisted pair offered a lot of advantages in terms of cost, and ease of running wires through the manufacturing floor and into everyone's office.

Getting the ArcNet deployed and working wasn't easy. All the PCs had to have ArcNet cards installed, we set up hubs and learned about limitations of hub-hub communication, run lengths of wires, and all sorts of stuff that was not always entirely clear (or correct) in the manuals.

Not coincidentally, this was also the time I started hosting PC LAN gaming parties at my house. We always had some spare cards, wires, and hubs available, so I'd just borrow a few for the weekend. Friends would come to my house, we'd install the network cards, create an ad-hoc ArcNet network, and play games. I think it was Command and Conquer?

The other thing we did, was install software on the VAX that made it appear to be a Novell Netware server. This software was incredibly slow and used a lot of resources. But that was ok, because it meant our nightly processing could dump files to the VAX disk, and those files could be copied (at night) to the actual Novell server for use by users the next day.

The last big topic I want to address is early open source. DEC, the company who made the VAX, was the second largest computer maker in the world after IBM. They supported a user community called DECUS (I think DEC US user group?). DECUS wasn't _run_ by DEC exactly, but was obviously heavily influenced. There were local DECUS chapters all over, including in the Twin Cities. And an annual conference that I attended a couple times (my first real conferences!).

Perhaps most importantly, DECUS collected freeware, shareware, and open source, and made it available to member organizations (like ours) a couple times a year. This all came via magnetic tapes that I'd restore onto our developer VAX.

I was already hooked into this world by having an Amiga. The Amiga user community was strong, and Walnut Creek and Fred Fish and others were collecting freeware/shareware/open source software and providing it to people via floppy disk collections and eventually CDs. I took full advantage of those resources, sending in my money and getting disks in the mail with cool games, tools, programming languages, and all sorts of stuff.

The DECUS tapes were the same, but for the VAX. And I devoured much of this software and used it to great advantage for INSTAR and myself.

For example, uucp was software that provided a VAX implementation of UseNet. Leveraging the local DECUS user group, I found a sympathetic geek at a company who already had UseNet access, and he let us dial into their system to do UseNet exchange (this was a store-and-forward protocol). Basically UseNet was the immediate predecessor to what we think of was the Internet or Web today, and a successor to BBS systems that people ran on things like Commodore 64 computers.

I was able to get access to UseNet newsgroups for the VAX, for table-top RPG games, and all sorts of stuff. Mostly work related, but some fun of course 😉

The other big-impact software was awk. Well really gawk (the GNU implementation of awk), compiled to run on the VAX.

If you are unfamiliar, awk is a *nix command that does amazingly powerful text processing work, mostly on files. It uses regular expressions, has a limited programming language, and can be used to convert data in a file into another file that's completely different.

This was _transformative_ for INCSTAR when coupled with the Novell file system. I'd been trying to convince users that they didn't need to print these 6" tall stacks of paper every night, without success. But at this point in time pretty much everyone had a PC on their desk that had Excel (version 3c - and yes, the 'c' is important).

So what we started doing was running those big reports _to disk_ instead of to a printer. Then we'd run an awk script that would transform the report into a tab delimited file that could be opened in Excel. And once in Excel, the user could easily search, filter, and sort the results. More sophisticated users could do much more obviously.

This finally did cut down on how much we printed - radically - as users figured out just how nice it was to have this data in a spreadsheet instead of a stack of paper.

OK, last thing: the spreadsheet wars.

The few PCs that existed at INCSTAR to start with were mostly used to run Lotus 123 and WordStar and that sort of thing.

And then Microsoft came out with workable versions of Excel and Word. And along in here (and my memory isn't clear) Windows 3.0.

My recollection is that the industry (reading trade magazines) and INSTAR itself, went through this big back and forth between Excel and Lotus 123, and between various popular word processors and Word. As we all know, Microsoft eventually won, but it didn't happen overnight, and for people in IT at the time, it was far from painless!

What didn't help Microsoft was that Excel 1, 2, 3, 3a, 3b, and 3c weren't always backward compatible. Our users were pretty sophsiticated, not just in accounting, but also in the science, lab, and R&D areas. So they wrote macros, and called _us in IT_ when an update to Excel broke their macros. It was a real pain!

Ultimately Excel 3c, as I recall, actually worked and was stable, and we ran that version for a very long time!

To Microsoft's credit, I think they learned a lot from those early days, and the Office team's subsequent obsession with backward compatibility is, in my view, absolutely necessary!

As you can tell, I did a lot while working in IT operations, help desk, and that whole world. Yet I was always wishing for opportunities to build software, and when the guy who managed the app dev part of IT left INCSTAR, I immediately let Dan know I wanted that job.

That'll be my next post in this series.