---
layout: post
title: How I Got Into Computers University Edition
date: 2020-08-17T00:00:00.0000000-05:00
---
This is the second post in a series about how I got into computers and how my career has unfolded. [Part 1 was pre-university](https://blog.lhotka.net/2020/08/15/How-I-Got-Into-Computers).

I went to two universities: the US Air Force Academy in Colorado, and Bemidji State University in northern Minnesota.

At USAFA I did take a computer class; some sort of intro to computing where we used Turtle Graphics (logo?) to control a pen that drew shapes on the screen. I think we might have learned some Pascal as well.

I ended up leaving USAFA after a year and transferred to Bemidji State in Minnesota. With most of my non-major classes out of the way, I focused largely on a Computer Science major and Mathematics minor. The minor was kind of a no-brainer, because it was just 1-2 extra classes beyond the requirements for Computer Science.

My biggest regret about my university time, is that I didn't really take advantage of my _non-major_ classes. I did them because they were required get to the stuff I really wanted to learn. Now, decades later, I find myself very interested in things like philosophy and some aspects of history, and wish I'd put more energy into those topics when I had the chance.

BSU used DEC VAX computers (and so did USAFA) for their curriculum. The primary language used by the CS department was Pascal, specifically DEC Pascal, which was in many ways closer to Modula II than Pascal, in that we had access to a type of module concept, multiple compiled output units, and a linker that combined the compiled files into a final executable.

What is interesting, when I compare my youngest son's experience (he recently got a EE degree with a CS minor) to mine, is how much is the same, and what is different.

1. Both of us took basic programming and algorithms type classes
1. He learned object oriented concepts, which didn't exist yet when I was in school
1. He used a combination of Java and C#, where I used Pascal
1. We both had assembly language classes
1. I took a class on how to create an assembler, he did not
1. We both used command line interfaces and simple text editors (I used something called TPU (text processing utility) and he used vim)
   1. The fact that universities still use text editors rather than modern tools like sublime, vscode, eclipse, visual studio, etc. just boggles my mind!!
1. We both took a computer architecture class; how gates work, how memory is addressed, etc. For him this was an intro class and he went way deeper in subsequent classes, while for me this was the only "hardware" class I ever took
1. Both of us took ridiculous amounts of mathematics
1. Where he spent a lot of time in engineering classes, I spent my time in classes on how to write compilers, operating systems, numerical analysis, software modeling and simulations, and other high end CS content
1. Neither of us took a database class; in my case it was because relational data theory didn't have a class yet, and in his case because it wasn't required for the CS minor

That last point is, in my mind, a big sticking point. Even for me, with my degree predating relational database engines, the _concepts_ of data relationships were known and should have been taught. As I'll talk about in a future post, not knowing even basic data relationship concepts was a hardship!

What I think is most important from my time in school is this: you get out of it what you put into it.

BSU was not a big school, and some teachers were pretty iffy, and a couple were amazingly good (in particular, thank you Dave Miller!).

The thing is, I was doing what I loved. I was generally more interested in writing software than partying or doing a lot of other "normal college stuff". 

I wrote my own MUD (multi-user dungeon) called Mordecai. Inspired by the MUD I'd played in middle and high school. As a side bar, it turns out that there's a fairly direct line between me writing Mordecai (and it becoming very popular) and how I met the woman who's now my wife.

The thing is, I learned so much by writing and maintaining this game. Concepts that they didn't teach in class, like 

* how to build a large scale bit of software
* how to handle bug and feature requests
* how to debug something that became quite large
* how to read/write configuration and data files (even without understanding data relationships)
* basic natural language parsing
* actual applications for many of the algorithms and data structures I'd learned in class
* multi-user contention issues
* multi-user communication in a near realtime setting

Now I'm not saying everyone should write a MUD in university. But I do think there's value in writing _something real_, because most classes never have students write anything of size. And size brings complexity, and an understanding of how to deal with complexity. This is important, because in the "real world" most software is big and complex, and has multiple concurrent users and performance requirements and bug reports and feature requests.

That MUD isn't the only thing I wrote. I love table-top gaming (strategy, role-playing, etc.) and I wrote software to help run various games.

I'd also moved off-campus and would dial in to the school's VAX using a 1200 baud modem (so fast!!). The problem was, I had a Commodore 64 with a very poor VT100 terminal emulator. So I wrote my own terminal emulator using C, including my own font rendering to get 80 characters across the screen (the C64 itself had no such capability).

I also took every software language class offered, including those from the Business School (COBOL and RPG II). So I finished school knowing Pascal, FORTRAN, COBOL, RPG II, BASIC, and DCL (sort of like powershell or bash). And bits and pieces of some other languages from my numerical analysis and simulation classes.

I didn't _have to_ take all those language classes, and all my CS friends though it was really weird that I'd take classes in the business languages, because they weren't "real programming".

In the end, I come back to the idea that I threw myself into learning everything I could about computing and software, and I got a lot out of it. And I practiced _a lot_ by writing software for myself and others outside of the requirements for any classes.

All that time, in retrospect, was incredibly well spent, because many of the opportunities I've had in my career came about because I'd done or learned something few other people knew, and knowledge is power.