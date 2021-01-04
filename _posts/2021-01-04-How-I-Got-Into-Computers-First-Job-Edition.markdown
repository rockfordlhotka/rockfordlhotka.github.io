---
layout: post
title: How I Got Into Computers First Job Edition
date: 2021-01-04T00:00:00.0000000
categories: []
tags: []
featuredImageUrl: https://blog.lhotka.net/assets/2021-01-04-How-I-Got-Into-Computers-First-Job-Edition/pdp-11-hd.png
---
This is the fifth post in a series about how I got into computers and how my career has unfolded.

* [Part 1 was pre-university](https://blog.lhotka.net/2020/08/15/How-I-Got-Into-Computers)
* [Part 2 was university](https://blog.lhotka.net/2020/08/17/How-I-Got-Into-Computers-University-Edition)
* [Part 3 was an internship](https://blog.lhotka.net/2020/12/27/How-I-Got-Into-Computers-University-Internship-Edition)
* [Part 4 was my first job search](https://blog.lhotka.net/2020/12/29/How-I-Got-Into-Computers-First-Job-Hunt-Edition)
 
My first job was with a company called ACS: Advanced Control Systems (?). It was a small company, with a dev lead and five developers (including me). Our dev lead/boss was named Mark, and he was incredibly talented and smart, though not without his faults.

The company was the world leader in software to run concrete ready-mix companies. This included things like:

* Tracking delivery trucks
* Taking customer orders
* Inventory management
* Interfacing with accounting systems
* Interfacing with the machines that loaded concrete trucks with sand/water/cement/rock/etc.

This was all on a DEC VAX platform, though the original software was on a PDP-11, and the reason they hired me was because of my university background on the VAX. The VAX software was brand new and wasn't yet at parity with the older PDP software, and I was hired to help work on bugs and features on the VAX side of things.

Up to this point, Mark had (without exaggeration) single-handedly created the VAX software by working evenings and weekends for about 18 months. I assume he was paid well for all that work, at least I sure hope he was!

As a point of interest, all this software was based on a _physical device_ created by the owner of the company. That physical device was a huge band of metal between two upright metal posts, one of which had a motor to rotate the post. Sort of like this diagram:

![](/assets/2021-01-04-How-I-Got-Into-Computers-First-Job-Edition/truck-tracker.png)

This apparatus would be mounted on a wall, and magnets would be placed on the slowly rotating metal surface (represented by the blue "Truck" items in the diagram).

The whole point of this thing was so people in a concrete dispatch office could put a magnet on the board to represent a truck that was being loaded with product, and the metal surface would slowly move the magnetic marker across the wall. This would _very roughly_ allow the idea of tracking a truck as it went through the loading, traveling, waiting, pouring, washing, returning process.

The primary claim to fame of the software was to do this truck tracking way better than this mechanical marvel.

The state of the art at that time was that each truck had a Motorola radio device with 7 buttons, and each button corresponded to one of the states a truck could be in, including loading, traveling, waiting, etc. That radio device sent a signal to a receiver, and that receiver was connected to the VAX computer via a serial cable. A service (daemon in *nix terms) was always running, listening for inbound data from that serial port. Anything that came in was written to a file, which allowed the VT terminals used by users to update with the latest data.

Now keep in mind that I grew up in rural Minnesota. Our local ready-mix company had _a single truck_. So to me this whole idea of needing complex software made little sense. Until I got this job, where customers included the company that was pouring the concrete for the massive Los Angeles freeway system. _That_ company had thousands of trucks, with a contract that said they'd have a truck on station to pour every 30 seconds!

Even in the Twin Cities, there were (and are) several ready-mix companies that have large numbers of trucks. I spent quite a bit of time over the years with one of them, and they are regional, not just Minnesota. Their dispatch office is pretty cool, with status screens hanging from the walls, and each dispatcher having 2-3 screens on their desks. Kind of like a small NASA mission control vibe.

The software was originally created on PDP-11 computers with paper terminals, where the user typed on a keyboard, everything printed on paper, and the results printed on that paper. Though by the time I was hired everyone had moved to terminals where the text printed on an electronic screen, but still just like a typical DOS, PowerShell, or bash experience, where the text just printed and scrolled up out of sight.

There were a couple really major differences between the PDP-11 software and the VAX software. The first is that the PDP-11 software was unique for each customer. The PDP-11 had this big removable hard drive platter.

![](/assets/2021-01-04-How-I-Got-Into-Computers-First-Job-Edition/pdp-11-hd.png)

Literally, each customer's software was on its own removable platter. So when a developer was going to fix a bug or make an enhancement for a customer's software, they'd remove whatever platter was in the drive, and put in the customer's platter. When the software was sold to a new customer, someone (a sales person?) would figure out the most similar existing customer, and that existing customer's platter would be copied onto a new, blank platter, and become the new customer's codebase.

Bugs were fixed on a per-customer basis, and a bug fix for one customer pretty much never rolled out automatically to any other customer. You can imagine how tedious and time consuming _that_ would have been!

The new VAX software was a unified codebase that was configurable. Which meant that there were far more configuration screens than "real" screens. All the differences between all the existing customer codebases were (hopefully) covered by configuration options on configuration screens in the new VAX codebase. The goal (and outcome) was that all customers would run the same code, and when a bug was fixed, or a feature added, for one customer, that became available to other customers. Not that other customers got new _features_ automatically; those usually involved some sort of up-charge of course.

> Today these are called "feature flags", which is a whole sub-industry within the software world. Back then we just called them config screens. Either way, the end result is pretty much the same.

Secondly, the new VAX software was entirely different in terms of the user experience. It was built with the assumption of a VT52 terminal (later VT100, then VT220), and those terminals understood something called an _escape sequence_: a special sequence of characters that would put the cursor at a specific location on the screen, and could make text bold, italics, or underlined. Basically very simplistic HTML, though less human-readable.

It is also important to realize that VT terminals weren't real smart. Unlike a 3270 terminal or web browser, they couldn't really draw their own screen or collect user input to send back all at once. Instead, VT terminals display or act on each character as it comes from the computer in a stream, and every keypress by the user is immediately sent to the computer.

As a result, our software would, as a stream, send all the characters (escape sequences and text) necessary to "draw the screen", and you could watch the cursor flicker around the screen to put all the text where it needed to go. Also, as the user typed, _each individual keystroke_ had to be processed by the computer.

That last bit is pretty cool, because we were able to make the software entirely interactive. As the user was typing we parsed the input and when appropriate updated the screen with new data or other information. Very comparable (in a text format) to a Window GUI or modern web experience, where the user gets immediate and responsive interaction. Very _unlike_ the typical mainframe or early web experience where the user would enter all their stuff in a screen and send the entire screen's worth of data to the computer to be processed as a block or batch.

Which makes for the core of one of my favorite stories: how a "much improved" UI was a very serious problem. This new UI was much easier to learn and understand, because it wasn't just a series of prompts and responses scrolling up off the top of the screen like the old "UI". _However_, the user base had muscle memory: enter the customer id, press enter three times, type the product code, press enter twice, override the price, etc. For the most part, they never looked at the screen, instead poring through the printed book of customers, products, etc. they kept on their desks.

Our new fancy UI meant they didn't necessarily need those old printed customer and product lists, because the new UI could do in-place look-ups and more. The thing is, the end users rebelled against the new UI. They'd been using the old one for a long time and had no interest in learning this new fancy crap. It was less productive, and so had to go. Open rebellion (read: bad and loud customer feedback directly to our management) rapidly led to change.

That change? My second big task in my new job was to go through the primary UI screens and make it so that end users could enter the customer id, press enter three times, type the product code, press enter twice, override the price, etc. In other words, the _default_ behavior was to skip over numerous fields in the pretty UI, so the user would effectively end up on the same prompts with the same keystrokes as the original scrolling UI.

I've never underestimated the value of the UI, UI design, or minimal keystrokes (or mouse usage) ever since my very first job! Anyone who thinks this aspect of software is not central to success clearly never had to deal with a bunch of concrete truck drivers who injured their backs, and thus were stuck in a sh-tty office job as a dispatcher, and who were pissed off because some computer yahoo randomly changed the user experience to "make it better".

This does beg the question, of course: what was my _first_ task in my new job? This is another great story, also driving home the importance of the user experience.

The system would do work, and then ask the user for permission to continue. This was a common, recurring pattern throughout the UX. And when this happened, the user would see a big prompt on the screen saying "Press any key to continue".

I am not kidding when I tell you that the single most common call the help desk had to answer was end users being unable to find the "any" key on the keyboard. Remember, this is the late 1980s, and our primary users were concrete delivery truck drivers who were now stuck in an office job. But really, around 1990, _most people_ didn't know how to type, and really didn't know their way around a keyboard.

So my task was to go through the entire software package and replace "Press any key to continue" with "Press <Enter> to continue". The VT52 and VT100 keyboards actually _had_ a key named "Enter", and this change totally eliminated the primary question that was generating calls to the help desk.

Seriously!

This really was a good task for me to start, because I had to go through pretty much every screen and module in the entire codebase to find this text, and so it was a good way to get some basic understanding of the way the software was structured.

Oh, yeah, I haven't mentioned one of the main things that gave me pause before accepting this job! The software was written in a programming language called VAX Basic. In my younger years, like so many computer science people, I was very biased about languages. If it wasn't C or Pascal, it had to pretty much suck. And, of course, _BASIC_ was at the bottom of the heap!

> So why did I even take a job programming BASIC? Because I really, really needed a job. That first job, regardless of language or industry vertical, is _so incredibly important_, because it gets you that 1-2 years of experience necessary to be considered for other jobs!

Little did I know, until I got into it, that _VAX Basic_ was amazing! It had all the goodness of FORTRAN and Pascal (and later I learned, a lot in common with Modula II). This was an eye-opening experience to me, and started to counter my ideological assumptions and biases around programming languages.

In fact, I totally fell in love with VAX Basic, and remember it fondly to this day. There are features of that language that I still miss today in C#, such as what C calls union structs: the ability to define multiple overlays for the same chunk of memory. And I highlight this specific feature because it features prominently later in my career.

The concrete dispatching software used that capability extensively to implement data access. The VAX operating system (VMS, and later OpenVMS) included a rich file system called RMS, and RMS natively supported "indexed files", which are files that are a blob, where the first n characters are an index. They were also known as ISAM files on other platforms, and in today's world they are extremely common in cloud environments such as Azure and AWS.

When using such an indexed blob file system, you need an efficient way to create and consume a blob of data (record), where the first n characters of the blob is the key. I don't remember the syntax from 30+ years ago, but the basic idea is in this pseudocode:

```c#
  public struct CustomerRecord
  {
      int Id;
      string[30] Name;
      string[100] Address;
  }
  
  public struct CustomerOverlay overlays CustomerRecord
  {
      byte[4] KeyData;
      byte[130] RecordData;
  }

...

  var customer = new CustomerRecord();
  customer.Id = 42;
  customer.Name = "Ahmed Johnson";
  customer.Address = "123 Somestreet, Gallifrey";
  
  var data = new CustomerOverlay(customer); // no data copied!!
  Database.Save(data.KeyData, data.RecordData);
```

On the VAX, FORTRAN and VAX Basic both had the ability to take two struct definitions like that and "aim them" at the exact same memory. So without any overhead of copying data or mapping data or anything, you could load the `Id`, `Name`, and `Address` values, and then get the whole thing as a single 134 character blob using the 'KeyData' and `RecordData` fields. Super fast and extremely powerful!

The entire software system was built around this concept for persisting data.

Also, it is worth noting that this all pre-dates relational database engines or SQL. Or at least in my experience it does; if any SQL databases existed in the late 1980s they weren't on my radar, or known to anyone at ACS.

As a result, any "relational" behaviors between different files were implemented entirely in our code. And we did use at least what I later learned was called third normal form for all our data. An important concept that wasn't taught in my university experience, but which was critical for success in my career.

OK, I can see that this has rambled on a bit. Let me focus a bit on lessons learned.

First, in retrospect, this job was one where _I built the product_. As opposed to later jobs where I supported people building the product, or where I was the product (topics for future posts). It is pretty nice to be someone creating the product that makes your company its money. At least, it can be, if you can get into a position where you are able to help shape the product, and I did have that opportunity. If you _don't_ get into that sort of position, I suspect this sort of job becomes a lot like assembly line work in a factory, where you are just coding to a spec written by someone who does have influence.

I came into this company at the perfect time in this regard, because I was hired to help "finish" the VAX version of the software, and I took to the whole thing like a duck to water. As a result, after working there for a year, I was able to be the lead designer and developer for a related product focused on aggregate (asphalt, etc.) as the company expanded beyond concrete.

Looking back on it, that was pretty amazing for someone just out of school, with literally one year's experience, to build a whole product for a new market segment.

Second, I learned a lot from Mark (the dev lead and architect). Some good, and some bad.

I came into this job thinking "code is art", and I still do believe that to a degree. But Mark had _very strict_ coding standards, and he'd dial back into work at night (via 1200 baud modem) and review my code. When I'd come in the next morning, he'd have red-lined printouts of things I needed to fix to conform to the standards.

I _railed_ against this oppression! The thing is, _most_ of his standards were backed by solid reasoning, and he took the time to defend them to me, allowing me to learn the "why" of those standards. Other parts of the standard were just arbitrary, and to this day I think kind of silly. Like all indenting was one tab and two spaces. What?!?

> Though I guess these days, one could argue that Mark dodged the whole tabs vs spaces argument by using both? Just irritate both sides equally?

Eventually I accepted the standards. There are two things about that I'd like to convey.

One is that I've since learned, from artist friends, that art requires constraints. Artists choose a medium and other constraints, and then work within those boundaries. Coding standards are a set of constraints, _within which_ you can practice the art of code. At least that's my view, as I still think of code as, at least in part, being art.

The second, is that about a year into my employment, ACS was purchased by an Alabama company called Command Data, and we became ACS/Command Data. Command Data was the world leader in dedicated _accounting_ software for ready-mix and aggregate companies, and so was very complimentary to ACS, which did the dispatching, ordering, truck tracking, etc. Also, they both used DEC VAX software and VAX Basic. So there was a lot of business and technical compatibility.

The thing is, Command Data didn't have Mark prior to that point. And it turns out that VAX Basic _can be made to look like_ FORTRAN, COBOL, Applesoft BASIC, and who knows what else. Every developer at Command Data used their own coding style and conventions, and none of their code looked anything like anyone else's code. I am serious when I say that one had code that looked like FORTAN, another like COBOL, and another like Applesoft (with line numbers and everything!).

This was a great job security thing, because nobody could _maintain_ anyone else's code either!

This really drove home, to me, the immeasurable value of coding consistency. Coding standards, and beyond that, consistent architecture and implementation of things like persisting data, interacting with the user, and more.

Painful as it was for a while, Mark taught me the value of coding standards and consistency, a lesson I've held dear ever since.

Sadly, Mark was also in the habit of taking credit for other people's work. We developers would do some pretty neat stuff, and over time I noticed that all that cool stuff was viewed as something _Mark did_ by the business people in the company. That stung, sometimes a lot! Another lesson I've held dear ever since: when you are someone's boss, make sure they are recognized for what they've done.

There are so many other stories in my mind from my relatively short time (less than two years) at ACS/Command Data. I learned so much about life in cube-land, the basic mental requirements of working 8-5 every weekday forever (in contrast to the largely self-directed time management in university), the need to bring your own lunch to avoid wasting all your money eating out, the pain of commuting and how to mentally deal with it.

I think this is why organizations are reluctant to hire people right out of school. It takes a good 3-6 months for someone who's never worked a full-time job to internalize all the basic skills necessary to function in cube-land, communicate with bosses and coworkers, plan ahead for communiting, etc.

There are life lessons I learned too. For example, my wife and I got married during this time, and moved from a nice apartment to a cheaper and less nice location, because we were slowly going broke due to rent. As a student I was very tight with money, but expenses were very predicable. As a young professional, expenses are less predictable, and higher. Nicer clothes, the need for regular haircuts, and all the things necessary to function in cube-land cost money, so there's a fair amount I had to learn about those costs and how to budget for them.

There was one time when I forgot my paycheck on my desk at work (this predates direct deposit). The thing was, we had bills due, no money in the bank, and virtually no gas in the car. We drove back to my office, got the check, and _almost_ made it home before the car ran out of gas! Every time I see a car stalled on the highway/freeway I think of my own personal experience: hiking up to an exit, finding a store, calling a coworker, waiting for him to come help us out. All so we could deposit the paycheck so we could pay the bills (and buy more gas).

That all changed when we moved to Alabama with the Command Data acquisition. 

> Most of the ACS employees didn't move to Alabama. I think three of us (Mark, myself, and Tim, another developer and friend) made the move. Others either weren't invited, or didn't want to make the move.

The cost of living in Birmingham, AL was (and I think still is) _much, much lower_ than in the Twin Cities. It was like getting a massive raise, and changed our lives dramatically, because we stopped eating away at savings, and started _increasing_ our savings!

After a few months of living as newlyweds in Alabama (which is a whole other set of stories), we both started to get homesick for family. We were thinking about what life would be like if we had kids so far from grandparents, and all sorts of similar things. Also, the culture differences between Alabama and Minnesota should never be underestimated. The Deep South is a thing unto itself, and we struggled to adjust.

So I started looking for jobs back in Minnesota. Fortunately I was still in touch with some of the ex-ACS employees that didn't get moved to Alabama, and one of them was aware of a couple programmer jobs at his new employer. Interestingly enough, my friend Tom was also looking, and we _both_ ended up getting those programmer jobs back in Minnesota!

And that will be another post in this series.