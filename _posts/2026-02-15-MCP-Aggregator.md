---
layout: post
title: MCP Aggregator
postDate: 2026-02-15T12:00:00-06:00
categories: []
tags: []
published: true
permalink:
image: /assets/2026-02-15-MCP-Aggregator/aggregator.png
---

![MCP Aggregator](/assets/2026-02-15-MCP-Aggregator/aggregator.png)

If you've been using AI coding tools like Claude Code, Cursor, or GitHub Copilot, you've probably started connecting them to MCP servers. One server for your database, another for your docs, maybe one for your company's internal APIs. It works great — until you have five or six servers configured, each tool needs its own copy of the configuration, and every server connection is sitting open whether you're using it or not.

That's the problem I set out to solve with [mcp-aggregator](https://github.com/MarimerLLC/mcp-aggregator).

## The Problem with Many MCP Servers

The Model Context Protocol is fantastic for giving AI tools access to external capabilities. But the current model has each AI tool maintaining its own direct connection to every MCP server you want to use. This creates a few headaches:

1. **Configuration sprawl** — Every tool (Claude Code, Cursor, VS Code, etc.) needs its own copy of every server's connection details. Add a new server? Update the config in every tool. Change a server's address? Update everywhere again.
2. **Resource waste** — All those connections sit open even when the AI isn't using them. Most of the time your AI tool is calling one or two servers, but all of them are consuming resources.
3. **No central management** — There's no single place to see what servers are available, add new ones, or remove old ones without touching every tool's configuration.

## One Connection to Rule Them All

The mcp-aggregator acts as a gateway between your AI tools and all your MCP servers. Instead of each tool connecting to every server directly, each tool connects to the aggregator. The aggregator manages all the downstream server connections.

Your AI tool configuration goes from this:

```json
{
  "mcpServers": {
    "database": { "command": "..." },
    "docs": { "command": "..." },
    "internal-api": { "command": "..." },
    "jira": { "command": "..." }
  }
}
```

To this:

```json
{
  "mcpServers": {
    "aggregator": { "url": "http://localhost:8080/mcp" }
  }
}
```

One connection. All your servers are still available, but the aggregator handles the routing.

## Lazy Loading and Idle Timeout

One of the design decisions I'm most pleased with is lazy loading. The aggregator doesn't connect to a downstream server until someone actually calls one of its tools. If you have ten servers registered but only use two during a session, only two connections get opened.

And connections don't stay open forever. After a configurable idle timeout (30 minutes by default), unused connections are automatically closed. They'll reconnect seamlessly the next time a tool is invoked.

This means the aggregator is efficient by default, without any manual connection management.

## Dynamic Registration

You can add and remove MCP servers at runtime without restarting anything. The aggregator exposes both MCP tools and a REST API for server management, so you can register a new server with a simple API call or even have your AI tool do it for you.

This is particularly useful in team environments where servers come and go, or when you're experimenting with new MCP servers and don't want to restart your AI tools every time.

## Skill Documents

One feature that emerged from real usage is skill documents. These are markdown files that describe _when and how_ to use a particular server's tools. Think of them as instructions that help the AI make better decisions about which server to call for a given task.

When an AI tool asks the aggregator what's available, it gets back concise summaries. When it needs more detail, it can drill down into a specific server's full tool schemas or read its skill document. This two-step discovery process keeps the initial response small while still providing rich information when needed.

## Built with .NET

The aggregator is built with .NET and is available as both a stdio server (for direct integration with tools like Claude Code) and an HTTP server (for shared/networked deployments). There are also Kubernetes manifests if you want to run it in a cluster.

The project is open source under the MIT license at [github.com/MarimerLLC/mcp-aggregator](https://github.com/MarimerLLC/mcp-aggregator). If you're juggling multiple MCP servers across multiple AI tools, give it a try.
