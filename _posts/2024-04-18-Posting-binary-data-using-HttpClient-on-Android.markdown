---
layout: post
title: Posting binary data using HttpClient on Android
date: 2024-04-18T00:00:00.0000000-05:00
categories: []
tags: []
published: true
permalink: 
featuredImageUrl: 
---
There is an issue folks have encountered when using the .NET `HttpClient` service to do a `POST` to an ASP.NET Core controller, where the payload of the post is binary data.

For example:

```c#
      var client = new HttpClient();
      var content = new ByteArrayContent(serialized);
      var httpResponse = await client.PostAsync(
        "<server url>",
        content);
```

The result is an exception: `Unable to read beyond end of stream`

It turns out that the problem is that the default `HttpClient` configuration on Android sends the data over the network with the wrong HTTP `Content-Type` header.

The `Content-Type` should be either `application/octet-stream` or left off completely since that is the default.

Instead, what is sent is `application/x-www-form-urlencoded`. Totally invalid in this context.

As a result, the ASP.NET Core server doesn't properly handle the binary payload, because it is following the instructions based on the incorrect `Content-Type` header.

How do you solve this?

Configure the `HttpClient` service differently on the client:

```c#
  var client = new HttpClient(new SocketsHttpHandler()));
```

The `SocketsHttpHandler` properly handles the `ByteArrayContent` content object and sends the correct header to the server.

Because I think this is a bug in the default `HttpClient` implementation on Android, I've filed a bug:

https://github.com/dotnet/maui/issues/21933