---
layout: post
title: What is an MCP?
postDate: 2025-12-05T18:02:53.3750407-06:00
categories: []
tags: []
published: true
permalink: 
featuredImageUrl: /assets/2025-12-05-What-is-an-MCP/mcp.png
---
Recently I've been fielding a number of questions about MCP, or Model Context Protocol. So I thought I'd write a quick post to explain what it is, and why it's important.

When someone asks me "what is an MCP", it is clear that they aren't asking about the MCP protocol, but rather what an MCP server is, and why it matters, and how it can be implemented. Does it need to be AI, or just used by AI?

![What is MCP?](/assets/2025-12-05-What-is-an-MCP/mcp.png)

At its core, MCP is nothing more than a protocol like REST or GraphQL. It's a way for different software systems to communicate with each other, specifically in the context of AI models. It is a client-server style protocol, where the client (often an AI model) makes requests to the server (a tool) to perform specific actions or retrieve information.

Another way to look at it, is that "an MCP Server" is an endpoint that exposes this MCP protocol. The server can be called by any client that understands the MCP protocol, including AI models.

Yet a third way people talk about MCP is that "an MCP" is an implementation of some software that does something useful - which happens to expose its behavior as an MCP Server endpoint via the MCP protocol.

## A Tiny Bit of History

Not long ago (like 2 years ago), AI models had no real way to interact with the "real world". You could ask them questions, and they could generate a response, but they couldn't take actions or access up-to-date information.

About two years ago, OpenAI introduced the idea of "function calling" in their API. This allowed developers to define specific functions that the AI model could call to perform actions or retrieve information. This was a big step forward, but it was still limited in scope.

Around the same time, other companies started to explore similar ideas. For example, LangChain introduced the concept of "tools" that AI models could use to interact with external systems. These tools could be anything from simple APIs to complex workflows.

Building on these ideas, the concept of MCP emerged from Anthropic as a standardized way for AI models to interact with external systems. MCP defines a protocol for how AI models can make requests to tools, and how those tools should respond.

### COM unication

Excuse the poor pun, but to me, MCP feels a lot like the old COM (Component Object Model) protocol from Microsoft days of old. COM was a way for different software components to communicate with each other, regardless of the programming language they were written in.

Like COM, MCP has an `IUnknown` entry point called `tools/list`, where clients can query for a list of available tools. Each tool is basically a method that can be called by the client. The client can also pass parameters to the tools, and receive results back.

In this post though, I really don't want to get into the details of the MCP protocol itself. It is enough to know that it is a client-server or RPC (Remote Procedure Call) style protocol that allows AI models to make imperative (and generally synchronous) calls to external systems in a standardized way.

## Why is MCP Important?

What most people _really_ want to know, is "what is an MCP" in the context of building AI systems. Why should I care about MCP?

An MCP server should be viewed as an API that is designed to be called by AI models. The idea is to allow an AI model to do things like:

- Access up-to-date information (e.g. weather, news, stock prices)
- Perform actions (e.g. book a flight, send an email)
- Interact with other software systems (e.g. databases, CRMs)

The goal is to provide AI models with a way to access current information, and to take actions in the real world. This is important because it allows AI models to be more useful and practical in real-world applications.

This changes an AI from being an isolated "chatbot" into the center of a broader system that can interact with the world around it.

## MCP vs Traditional APIs

The trick here, is that an MCP server _is not the same as a traditional API_.

A traditional API is designed to be called by other applications. Often this means that the API is designed with a specific use case or workflow in mind. The client application is expected to know how to use the API, and what data to send and receive. In most cases, it is assume that the client software will be written to call different API methods in the correct order, and handle any errors or exceptions that may occur.

An MCP server, on the other hand, is designed to be called by an AI model. This means that the API needs to be designed in a way that is easy for the AI model to understand and use. The AI model may not have any prior knowledge of the API, so the API needs to be self-describing and intuitive.

It also means that the client (the AI model) is non-deterministic, or probabilistic. The AI model may not always call the API methods in the correct order, or may send unexpected data. The MCP server needs to be able to handle these situations gracefully, and provide useful feedback to the AI model.

Can you imagine if you took an inexperienced human and handed them a traditional API spec, and expected them to use it correctly? That's what it's like for an AI model trying to use a traditional API.

## Implementing an MCP Server

Implementing an MCP server involves a few key steps:

1. Figure out the scope and purpose of the MCP server. What actions will it need to perform? What information will it need to provide?
2. Define the tools and methods that the MCP server will expose. This involves identifying the actions that the AI model will need to perform, and designing the API methods to support those actions.
3. Implement the MCP protocol. This involves creating the endpoints that will handle requests from the AI model, and implementing the logic to process those requests and return results.
4. Test the MCP server with the inspector and then an AI model.
5. Iterate and improve. Based on feedback from the AI model, you may need to refine the API methods, improve error handling, or add new features to the MCP server.

### Determining the Scope

As I write this, the MCP specificiation just turned one year old. As a result, there are not a lot of established best practices for designing MCP servers yet. However, I think it's important to start with a clear understanding of the scope and purpose of the MCP server.

![MCP birthday](/assets/2025-12-05-What-is-an-MCP/mcp-birthday.png)

We can hopefully draw on established principles from Domain Driven Design (DDD), object-oriented design, component-based design, and microservices architecture to help guide us.

All of these, properly done, have the concept of scope or bounded context. The idea is to define clear boundaries around the functionality that the MCP server will provide, and to ensure that the API methods are cohesive and focused.

For example, an MCP server might be designed to manage a specific domain, such as booking travel, managing customer relationships, or processing payments. The API methods would then be focused on the actions and information relevant to that domain.

### Tools and Methods

Once the scope is defined, the next step is to identify the tools and methods that the MCP server will expose. This involves breaking down the functionality into discrete actions that the AI model can perform. The granularity of these tools will depend on the specific use case and the needs of the AI model. In general, I would expect them to be at a higher level of abstraction as compared to a traditional API or microservice.

For example, instead of exposing low-level CRUD operations for managing customer data, an MCP server might expose higher-level tools for "Create Customer Profile", "Update Customer Preferences", or "Retrieve Customer Purchase History". These tools would encapsulate the underlying complexity of the operations, making it easier for the AI model to use them effectively.

### Implement the MCP Protocol

Actually implementing the MCP protocol by hand would involve understanding JSON-RPC and how it is used by MCP. Fortunately, there are packages available for most programming platforms and languages that already implement the protocol, allowing developers to focus on building the actual tools and methods.

The most obvious choice is the [ModelContextProtocol](https://github.com/modelcontextprotocol) packages on GitHub.

Rather than learning and implementing the protocol itself, use packages like these to focus on building the server and tools.

### Testing

Testing an MCP server directly from an AI model can be challenging, especially in the early stages of development. Fortunately, there are tools available that can help with this process.

A common option is the [MCP Inspector](https://github.com/modelcontextprotocol/inspector). The MCP Inspector is a tool that allows developers to interactively test and debug MCP servers. It provides a user interface for exploring the available tools and methods, making requests, and viewing responses.

Once you know that your MCP server is working correctly with the Inspector, you can then test it with an actual AI model. This will help ensure that the AI model can effectively use the MCP server to perform the desired actions and retrieve the necessary information.

### Iterate and Improve

The odds of getting the MCP server and its tools right on the first try are very low. Remember that your _consumer_ is an AI agent, which is probabilistic and is in some ways more like a naïve human than a traditional software application.

![Iteration](/assets/2025-12-05-What-is-an-MCP/iterate.png)

Once you are able to interact with the MCP server using an AI model, you will likely discover areas for improvement. This could include refining the API methods, improving error handling, or adding new features to better support the needs of the AI model.

In particular, look at:

- Ways to improve the descriptions of the tools, methods, and parameters to make them clearer and more intuitive for the AI model.
- Adding examples of how to use the tools and methods effectively.
- Enhancing error messages to provide more useful feedback to the AI model when something goes wrong.
- Changing the output of the tools to be more (or less) structured, depending on what the AI model seems to handle better.

### Logging

I didn't mention this in the steps above, but logging is very important when building an MCP server. You need to be able to see what requests are being made by the AI model, what parameters are being sent, and what responses are being returned.

These days most good server systems use open telemetry for logging and tracing. This is a good idea for MCP servers as well, as it allows you to collect detailed information about the interactions between the AI model and the MCP server.

As with any logging, be mindful of privacy and security concerns. Avoid logging sensitive information, and ensure that any logged data is stored securely.

## Security and Identity

Similarly, I didn't mention security and identity in the steps above, but these are also very important considerations when building an MCP server.

Usually this occurs at a couple levels:

- Is the client authorized to access the MCP server at all?
- Does the client represent a specific user or identity, and if so, what permissions does that identity have?

Of every topic in this post, security and identity is the most complex, and the one that is most likely to evolve over time. As of this writing (end of 2025), there are few established best practices for handling security and identity in MCP servers.

## What is in an MCP Server?

Everything I've said so far is great, but still doesn't answer one of the underlying questions: is an MCP server AI, or just used by AI?

The answer is: it depends.

![AI calling MCP](/assets/2025-12-05-What-is-an-MCP/ai-mcp.png)

### Getting Data

In many cases, an MCP tool be be used by the AI model to get up to date information from external systems. In these cases, it is probably best to write traditional code that accesses databases, APIs, or other data sources, and exposes that data via an MCP server. Because context is critical for AI models, a tool like this should provide the requested data, often with additional context to help the AI model understand how to use the data.

#### Retrieval-Augmented Generation

You may encounter the term "RAG" or retrieval-augmented generation in this context. This is a technique where an AI model retrieves relevant information from an external source (like a database or document store) to provide context for generating a response. An MCP tool that provides up-to-date information can be a key part of a RAG system. Technically RAG uses AI models to do the work, but these AI models aren't what most of us think of as "AI systems". They are specialized models that are focused on encoding text into vector arrays.

### Performing Actions

In other cases, an MCP tool might change or update data in external systems. Again, this is probably best done with traditional code that performs the necessary actions, and exposes those actions via an MCP server. Remember that most traditional APIs are not designed to be called by AI models, so the MCP server should provide a higher-level abstraction that is easier for the AI model to understand and use. Here too, context is critical. Usually this context comes from the descriptions of the tools, methods, and parameters.

### Using AI Inside an MCP Server

In some cases though, an MCP server might be implemented using AI itself. For example, you might expose some complex functionality to other AI models, where the logic is too complex to implement with traditional code. In these cases, you might use an AI model to process the requests and generate the responses.

For example, you might have a business process that involves complex and specialized knowledge and judgement. In this case, you might use an AI model tuned for that specific domain. This allows a non-specialized AI model, like a chatbot or something, to request that the specialized "sub-agent" perform the complex task on its behalf.

#### Specialized Sub-Agents

There are different techinques for building a specialized AI model for use in an MCP server implementation. These include:

- Using prompt engineering (including system prompts) to guide the model's behavior.
- Implementing retrieval-augmented generation (RAG) to provide the model with access to relevant information.
- Fine-tuning a base model with domain-specific data.

I've listed these techniques in order of increasing complexity. Prompt engineering is the simplest, and fine-tuning is the most complex.

In most cases, I would recommend starting with prompt engineering, and only moving to more complex techniques if necessary. The goal is to provide the AI model with enough context and guidance to perform the desired actions effectively.

## Conclusion

When someone asks me "what is an MCP", they are wondering what an MCP server is, and why it matters, and how it can be implemented.

In this post I've tried to answer those questions, and provide some guidance. At least as we understand such things at the end of 2025.
