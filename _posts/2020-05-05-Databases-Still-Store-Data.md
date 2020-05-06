---
title: Databases Still Store Data
weblogName: Rockford Lhotka
postDate: 2020-05-05
---
OK, I know the title of this post is patently obvious.

I recently [tweeted a thought about databases](https://twitter.com/RockyLhotka/status/1257395928413212673?s=20) though, and it generated some fun comments.

![]({{ site.url }}/assets/2020-05-05-Databases-Still-Store-Data/db-tweet.png)

The thing is, my career started before relational databases were popular, maybe before they were even a thing, though I'm not sure.

My first exposure to an RDBMS was around 1991, give or take. I installed Rdb on our DEC VAX to give the idea a try, only to find that running it would require hundreds of thousands of dollars in hardware (per year) to actually run the database, much less any software to use it.

Before that I'd had some years of experience using indexed files, a feature built into the file system of the VAX OpenVMS operating system, and a feature I've missed ever since switching to Windows.

I'd worked on and built software to run entire companies using indexed files. ERP software, inventory management, document management, heavy truck dispatching and logistics, point of sale.

> I was fortunate early in my career to work at a couple smaller companies where we had to build apps for nearly every aspect of each business. A gold mine of experience!

My point is that you don't need an RDBMS to build enterprise solutions. They are _nice_, but not required.

I also had an opportunity to use a hierarchical database on the VAX. I don't remember, and can't find, the name of the database, but it was built directly on the indexed file system and so had nearly no overhead. Certainly nothing compared to Rdb!

My next exposure to an RDBMS was Access on Windows 3.0. This wasn't a bad experience, and in 1993 when VB 3.0 came out with access to the underlying JET database (the engine underneath Access at the time), my team and I built quite a number of apps.

The thing I want to call out here, is that the RDBMS _concepts_ of normalizing data weren't new to me. We'd been using them since my first introduction to the indexed file system around 1987.

People seem to conflate normalizing data (or intentionally not normalizing it) with relational databases. But these are _not the same thing_! Normalizing data is a much older concept, and relational databases just implemented pre-existing patterns.

Server-based RDBMS implementations provide tremendous benefits of course! Most notably ACID transactions, journaling, rollback capabilities - all translating into data integrity and consistency.

Relational databases like FoxPro, Access, dBase, and the whole host of indexed file systems that came before laid the groundwork, but without a physical server process it was hard (impractical?) to get the level of integrity and consistency provided by a real database server.

It is also the case that by the mid-1990's relational database engines had become efficient enough that they weren't prohibitively expensive to operate. You could run Sybase or SQL Server on a high end PC, and Oracle on a low end super-server (ok, that was a cheap shot :) ).

I'm just pointing out that in ~1991 Rdb couldn't run on the VAX computer that supported a 300 employee company, and by ~1996 it was realistic to run SQL Server on a comparatively affordable PC. Part of that is Moore's Law, economic factors, and major improvements to database engines.

From the mid-1990's forward, like most people, my career flowed comfortably forward with the assumption that data was in some RDBMS server. Sure, there were performance comparisons and whatnot - Oracle vs SQL Server vs various other competitors. But at the end of the day it was a flavor of SQL running against an RDBMS engine on a server.

I honestly can't recall the first time I played with an "object database", but I want to say maybe around 2005? 2003?

The idea was to have a database that helped address the object-relational mapping problem by eliminating the relational part of the equation. 

> This problem space is laid out nicely in [Object Technology: A Manager's Guide](https://www.amazon.com/Object-Technology-Managers-David-Taylor/dp/0201309947).

The database I used was a .NET thing, intended to easily store and retrieve objects from a server. I went into it with a naive mindset, thinking I could just use good OO design principles, store and retrieve domain types without thinking about data relationships. Turns out that's not how this works!

Remember how I said data relationships like normalization pre-date RDBMS implementations? Yeah, there's a reason for that! There's no magic technology that lets us escape how data relates to itself.

> [Thomas Weiss explains NoSQL data modeling](https://myignite.techcommunity.microsoft.com/sessions/79932) in a super-understandable way in this video. I really wish this video had existed back then, as my attitude probably would have been very different!

At the time, disillusioned, I kind of ignored object databases, and document databases, and NoSQL, and all the stuff that flowed from those original object database concepts.

Until now.

I finally had a good reason to explore the use of MongoDb.

It turns out that the basic usage and concepts for MongoDb are directly analogous to that early object database I played with around 2004. Not a lot has changed in terms of how a developer interacts with a data store.

And, what caused my tweet, is the hard reality that the programmer API surface for interacting with database engines, on a surface level, aren't all that different. Indexed files, hierarchical databases, relational databases, document databases - they are all data in, data out.

Turns out databases just store data.

It also turns out that data relationships and data normalization exists in every kind of database implementation. Whether the database engine enforces those relationships for you, or how it does such a thing - that varies. But the _concepts_ of how data relates to itself, that's apparently universal.

As a result, the logical data model for an indexed file solution, a relational solution, or a document solution are quite similar. You do need to understand the basic philosophy behind each data storage mechanism to avoid major pitfalls, but at some level this is all pretty much the same stuff.

Maybe it is because I spent the formative years of my career with just indexed files, and we had to implement data relationships and integrity in our code, but all these concepts just seem like different faces of the same underlying thing.

So I stand by my tweet. Logical data models, data relationships, and the high level API surface area to implement create/read/update/delete (CRUD) operations on any kind of data store are pretty similar.

Before you get all riled up because I'm somehow dismissing whichever of these beautiful babies you happen to love, remember, I'm not saying they aren't different behind the scenes!

There are huge trade offs between ACID transactions, data consistency, global scaling, basic performance, and a whole host of other aspects.

I'm still, for example, struggling to see how document databases work efficiently for enterprise reporting needs. But it is immediately obvious how much more efficient they can be for many basic forms-over-data type business app scenarios.

Or how challenging it is for a relational database to manage sharding and global data distribute and scale. Things that are trivial for indexed files, and (though I haven't dug into it) are apparently part of the base fabric for document databases like CosmosDb.

So, what started as a tweet has now become the history of my use of data stores over the years, hopefully providing some insights into how little (and how much) has changed over time.
