---
title: What Architectures Were Used Before Microservices?
weblogName: Rockford Lhotka
postDate: 2019-12-29
---
Many architectures have been used over the decades. I’ll focus my answer on the 15 years or so immediately prior to today’s microservice fixation.

If you go back to the late 1990’s the primary architecture was n-tier client/server. This involved a client app talking to an app server that talked to a database. There were variations on this theme, but that was the core of n-tier.

In the early 00’s SOA (service-oriented architecture) gained a lot of hype.

There were some very different interpretations of what “SOA” actually meant, some of which were the same as today’s microservice architecture.

Other folks thought it meant defining one big master data management concept, with a small number of XML documents defining everything in an enterprise.

What really happened is that most folks ended up building n-tier client/server apps, just with radically less efficient technologies and protocols. XML, for example, is many times larger than the binary protocols being used in the late 1990’s. Fortunately hardware and networks got faster too, so most people never noticed the horrific waste caused by XML and later JSON - just to recreate n-tier client/server.

There is one other big thing that happened during this time: the web. In the late 1990’s a lot of apps were built using smart client technologies, so the client app talked to an app server. Throughout the 00’s this fell to maybe half, with the other half mostly building server-side web sites that used the client device (and browser) as a dumb terminal.

The arrival of the iPad really changed the game. At this point it became clear that smart client development was totally impractical, because it meant building client apps for Windows and iOS. As a result there was a major shift to web development, but so much disaffection with web site development that the web world embraced the idea of smart client development. Ultimately via Angular and React and TypeScript.

That largely brought us back to n-tier client/server, though now we tend to talk about it in terms of a SPA calling a set of “microservices” to get data. Objectively though, those “microservices” are no different than the n-tier app server services we were calling from smart clients in the 1990’s.

What I would call “real microservices” are a whole other thing, and are never exposed directly to a browser. Each service is a standalone app with its own interface (JSON or gRPC), its own business logic, and its own data. Services communicate with each other by sending messages; think email or the US Postal Service.

Generally speaking, if your services are being called from a browser or mobile device they aren’t a microservice. If your services do nothing but retrieve or save data, without any business logic, they aren’t microservices. If your services use point-to-point communication via HTTP, without regard for messaging concepts or auto-healing if a service goes down, they aren’t microservices.

What this means, in practice, is that most people, even today, even with with they call “microservices” are still building n-tier client/server apps, using the same basic architecture developed in the mid- to late-1990’s.

Though at least gRPC brings us back to a more efficient binary protocol than XML or JSON.