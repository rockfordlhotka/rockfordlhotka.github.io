---
title: What is the difference between SOA and microservices?
weblogName: Rockford Lhotka
postDate: 2020-01-31
---

There is no meaningful difference between SOA and “micro”-services in terms of architecture, benefits, and drawbacks. Basically “microservices” is just SOA rebranded, because SOA largely failed to gain adoption a decade ago.

Our industry does this cyclical repetition of the same concepts with new names constantly. And that’s OK.

The reason it is OK is that technology, process, and people’s expectations keep evolving over time. Sometimes it might take decades and numerous branding changes, but we usually do actually figure out how to make something work.

Back in the early 1990’s the ASP trend (application service provider) was basically a run at what we now call cloud computing. But it had to be rebranded and re-implemented several times before it actually succeeded.

The same with SOA and microservices. In the past decade a lot of things have changed and evolved. It might be that microservices will gain wide adoption. Or it might be that we need to wait another decade or two for more things to change.

What’s changed in the last decade that might make microservices work where SOA failed? Some examples:

1. Container-based deployment and cloud (private, public, hybrid) computing fabrics (AWS, Azure, Kubernetes, etc.)
1. More organizations have adopted agile
1. DevOps (or at least automated CI/CD with unit testing)
1. Languages like JavaScript/TypeScript and C# now have first-class constructs to deal with async programming (IO blocked, parallel, distributed)
1. Very few developers are left who haven’t implemented some successful distributed systems; mostly n-tier to be sure, but still, pretty much everyone understands distributed computing
1. The idea of a “master document” based definition of services was tried with SOA and (mostly) debunked, so hopefully we’re well past that silliness
1. Basically, technology has advanced, the adoption of process has advanced, and (I’d like to think) more people have a better understanding of the consequences of distributed computing and service-based architectures.

Yet, even with all that, I think it is most likely that microservices will not become mainstream, and that we have at least another decade of tech/process/people evolution to go before service-based systems become a mainstream success.

In other words I suspect that, like with SOA a decade ago, we’ll mostly all just re-implement n-tier client/server apps with the latest and greatest technology. And that isn’t a bad thing, because all these advances make n-tier development, deployment, and management far superior to where they were a decade ago, which was far superior to when n-tier went mainstream in the mid-1990’s.