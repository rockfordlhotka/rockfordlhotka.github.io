---
layout: post
title: A Simple CSLA MCP Server
postDate: 2025-10-02T15:39:53.3750407-05:00
categories: []
tags: []
published: true
permalink: 
featuredImageUrl: 
---
In a recent CSLA discussion thread, a user asked about setting up a simple CSLA Mobile Client Platform (MCP) server.

https://github.com/MarimerLLC/csla/discussions/4685

I've written a few MCP servers over the past several months with varying degrees of success. Getting the MCP protocol right is tricky (or was), and using semantic matching with vectors isn't always the best approach, because I find it often misses the most obvious results.

Recently however, Anthropic published a C# SDK (and NuGet package) that makes it easier to create and host an MCP server. The SDK handles the MCP protocol details, so you can focus on implementing your business logic.

https://github.com/modelcontextprotocol/csharp-sdk

Also, I've been reading up on the idea of hybrid search, which combines traditional search techniques with vector-based semantic search. This approach can help improve the relevance of search results by leveraging the strengths of both methods.

The code I'm going to walk through in this post can be easily adapted to any scenario, not just CSLA. In fact, the MCP server just searches and returns markdown files from a folder. To use it for any scenario, you just need to change the source files and update the descriptions of the server, tools, and parameters that are in the attributes in code. Perhaps a future enhancement for this project will be to make those dynamic so you can change them without recompiling the code.

The code for this article can be found in this [GitHub repository](https://github.com/MarimerLLC/csla-mcp).

> ℹ️ Most of the code was actually written by Claude Sonnet 4 with my collaboration. Or maybe I wrote it with the collaboration of the AI? The point is, I didn't do much of the typing myself.

Before getting into the code, I want to point out that this MCP server really is useful. Yes, the LLMs already know all about CSLA because CSLA is open source. However, the LLMs often return outdated or incorrect information. By providing a custom MCP server that searches the actual CSLA code samples and snippets, the LLM can return accurate and up-to-date information.

## The MCP Server Host

The MCP server itself is a console app that uses Spectre.Console to provide a nice command-line interface. The project also references the Anthropic C# SDK and some other packages. It targets .NET 10.0, though I believe the code should work with .NET 8.0 or later.

I am not going to walk through every line of code, but I will highlight the key parts.

> ⚠️ The modelcontextprotocol/csharp-sdk package is evolving rapidly, so you may need to adapt to use whatever is latest when you try to build your own. Also, all the samples in their GitHub repository use static tool methods, and I do as well. At some point I hope to figure out how to use instance methods instead, because that will allow the use of dependency injection. Right now the code has a lot of `Console.WriteLine` statements that would be better handled by a logging framework.

Although the project is a console app, it does use ASP.NET Core to host the MCP server.

```csharp
        var builder = WebApplication.CreateBuilder();
        builder.Services.AddMcpServer()
            .WithHttpTransport()
            .WithTools<CslaCodeTool>();
```

The `AddMcpServer` method adds the MCP server services to the ASP.NET Core dependency injection container. The `WithHttpTransport` method configures the server to use HTTP as the transport protocol. The `WithTools<CslaCodeTool>` method registers the `CslaCodeTool` class as a tool that can be used by the MCP server.

There is also a `WithStdioTransport` method that can be used to configure the server to use standard input and output as the transport protocol. This is useful if you want to run the server locally when using a locally hosted LLM client.

The nice thing about using the modelcontextprotocol/csharp-sdk package is that it handles all the details of the MCP protocol for you. You just need to implement your tools and their methods. All the subtleties of the MCP protocol are handled by the SDK.

## Implementing the Tools

The `CslaCodeTool` class is where the main logic of the MCP server resides. This class is decorated with the `McpServerToolType` attribute, which indicates that this class will contain MCP tool methods.

```csharp
  [McpServerToolType]
  public class CslaCodeTool
```

### The Search Method

The first tool is Search, defined by the `Search` method. This method is decorated with the `McpServerTool` attribute, which indicates that this method is an MCP tool method. The attribute also provides a description of the tool and what it will return. This description is used by the LLM to determine when to use this tool. My description here is probably a bit too short, but it seems to work okay.

Any parameters for the tool method are decorated with the `Description` attribute, which provides a description of the parameter. This description is used by the LLM to understand what the parameter is for, and what kind of value to provide.

```csharp
    [McpServerTool, Description("Searches CSLA .NET code samples and snippets for examples of how to implement code that makes use of #cslanet. Returns a JSON object with two sections: SemanticMatches (vector-based semantic similarity) and WordMatches (traditional keyword matching). Both sections are ordered by their respective scores.")]
    public static string Search([Description("Keywords used to match against CSLA code samples and snippets. For example, read-write property, editable root, read-only list.")]string message)
```

#### Word Matching

The orginal implementation (which works very well) uses only word matching. To do this, it gets a list of all the files in the target directory, and searches them for any words from the LLM's `message` parameter that are 4 characters or longer. It counts the number of matches in each file to generate a score for that file.

Here's the code that gets the list of search terms from `message`:

```csharp
        // Extract words longer than 4 characters from the message
        var searchWords = message
          .Split(new char[] { ' ', '\t', '\n', '\r', '.', ',', ';', ':', '!', '?', '(', ')', '[', ']', '{', '}', '"', '\'', '-', '_' }, 
                 StringSplitOptions.RemoveEmptyEntries)
          .Where(word => word.Length > 3)
          .Select(word => word.ToLowerInvariant())
          .Distinct()
          .ToList();

        Console.WriteLine($"[CslaCodeTool.Search] Extracted search words: [{string.Join(", ", searchWords)}]");
```

It then loops through each file and counts the number of matching words. The final result is sorted by score and then file name:

```csharp
        var sortedResults = results.OrderByDescending(r => r.Score).ThenBy(r => r.FileName).ToList();
```

#### Semantic Matching

More recently I added semantic matching as well, resulting in a hybrid search approach. The search tool now returns two sets of results: one based on traditional word matching, and one based on vector-based semantic similarity.

The semantic search behavior comes in two parts: indexing the source files, and then matching against the message parameter from the LLM.

##### Indexing the Source Files

Indexing source files takes time and effort. To minimize startup time, the MCP server actually starts and will work without the vector data. In that case it relies on the word matching only. After a few minutes, the vector indexing will be complete and the semantic search results will be available.

The indexing is done by calling a text embedding model to generate a vector representation of the text in each file. The vectors are then stored in memory along with the file name and content. Or the vectors could be stored in a database to avoid having to re-index the files each time the server is started.

I'm relying on a `vectorStore` object to index each document:

```csharp
  await vectorStore.IndexDocumentAsync(fileName, content);
```

This `VectorStoreService` class is a simple in-memory vector store that uses Ollama to generate the embeddings:

```csharp
    public VectorStoreService(string ollamaEndpoint = "http://localhost:11434", string modelName = "nomic-embed-text:latest")
    {
      _httpClient = new HttpClient();
      _vectorStore = new Dictionary<string, DocumentEmbedding>();
      _ollamaEndpoint = ollamaEndpoint;
      _modelName = modelName;
    }
```

This could be (and probably will be) adapted to use a cloud-based embedding model instead of a local Ollama instance. Ollama is free and easy to use, but it does require a local installation.

The actual embedding is created by a call to the Ollama endpoint:

```csharp
        var response = await _httpClient.PostAsync($"{_ollamaEndpoint}/api/embeddings", content);
```

The embedding is just a list of floating-point numbers that represent the semantic meaning of the text. This needs to be extracted from the JSON response returned by the Ollama endpoint.

```csharp
        var responseJson = await response.Content.ReadAsStringAsync();
        var result = JsonSerializer.Deserialize<JsonElement>(responseJson);
        
        if (result.TryGetProperty("embedding", out var embeddingElement))
        {
          var embedding = embeddingElement.EnumerateArray()
            .Select(e => (float)e.GetDouble())
            .ToArray();
          
          return embedding;
        }
```

> 👩‍🔬 All those floating-point numbers are the magic of this whole thing. I don't understand any of the math, but it obviously represents the semantic "meaning" of the file in a way that a query can be compared later to see if it is a good match.

All those embeddings are stored in memory for later use.

##### Matching Against the Message

When the `Search` method is called, it first generates an embedding for the `message` parameter using the same embedding model. It then compares that embedding to each of the document embeddings in the vector store to calculate a similarity score. All that work is delegated to the `VectorStoreService`:

```csharp
  var semanticResults = VectorStore.SearchAsync(message, topK: 10).GetAwaiter().GetResult();
```

In the `VectorStoreService` class, the `SearchAsync` method generates the embedding for the query message:

```csharp
      var queryEmbedding = await GetTextEmbeddingAsync(query);
```

It then calculates the cosine similarity between the query embedding and each document embedding in the vector store:

```csharp
        foreach (var doc in _vectorStore.Values)
        {
          var similarity = CosineSimilarity(queryEmbedding, doc.Embedding);
          results.Add(new SemanticSearchResult
          {
            FileName = doc.FileName,
            SimilarityScore = similarity
          });
        }
```

The results are then sorted by similarity score and the top K results are returned.

```csharp
        var topResults = results
          .OrderByDescending(r => r.SimilarityScore)
          .Take(topK)
          .Where(r => r.SimilarityScore > 0.5f) // Filter out low similarity scores
          .ToList();
```

##### The Final Result

The final result of the `Search` method is a JSON object that contains two sections: `SemanticMatches` and `WordMatches`. Each section contains a list of results ordered by their respective scores.

```csharp
        var combinedResult = new CombinedSearchResult
        {
          SemanticMatches = semanticMatches,
          WordMatches = sortedResults
        };
```

It is up to the calling LLM to decide which set of results to use. In the end, the LLM will use the fetch tool to retrieve the content of one or more of the files returned by the search tool.

### The Fetch Method

The second tool is Fetch, defined by the `Fetch` method. This method is also decorated with the `McpServerTool` attribute, which provides a description of the tool and what it will return.

```csharp
    [McpServerTool, Description("Fetches a specific CSLA .NET code sample or snippet by name. Returns the content of the file that can be used to properly implement code that uses #cslanet.")]
    public static string Fetch([Description("FileName from the search tool.")]string fileName)
```

This method has some defensive code to prevent path traversal attacks and other things, but ultimately it just reads the content of the specified file and returns it as a string.

```csharp
    var content = File.ReadAllText(filePath);
    return content;
```

## Hosting the MCP Server

The MCP server can be hosted in a variety of ways. The simplest is to run it as a console app on your local machine. This is useful for development and testing.

You can also host it in a cloud environment, such as Azure App Service or AWS Elastic Beanstalk. This allows you to make the MCP server available to other applications and services.

Like most things, I am running it in a Docker container so I can choose to host it anywhere, including on my local Kubernetes cluster.

For real use in your organization, you will want to ensure that the MCP server endpoint is available to all your developers from their vscode or Visual Studio environments. This might be a public IP, or one behind a VPN, or some other secure way to access it.

I often use tools like Tailscale or ngrok to make local services available to remote clients.

## Testing the MCP Server

Testing an MCP server isn't as straightforward as testing a regular web API. You need an LLM client that can communicate with the MCP server using the MCP protocol.

Anthropic has an npm package that can be used to test the MCP server. You can find it here:

https://github.com/modelcontextprotocol/inspector

This package provides a GUI or CLI tool that can be used to interact with the MCP server. You can use it to send messages to the server and see the responses. It is a great way to test and debug your MCP server.

Another option is to use the MCP support built into recent vscode versions. Once you add your MCP server endpoint to your vscode settings, you can use the normal AI chat interface to ask the chat bot to interact with the MCP server. For example:

```text
call the csla-mcp-server tools to see if they work
```

This will cause the chat bot to invoke the `Search` tool, and then the `Fetch` tool to get the content of one of the files returned by the search.

Once you have the MCP server working and returning the types of results you want, add it to your vscode or Visual Studio settings so all your developers can use it.

In my experience the LLM chat clients are pretty good about invoking the MCP server to determine the best way to author code that uses CSLA .NET.

## Conclusion

Setting up a simple CSLA MCP server is not too difficult, especially with the help of the Anthropic C# SDK. By implementing a couple of tools to search and fetch code samples, you can provide a powerful resource for developers using CSLA .NET.

The hybrid search approach, combining traditional word matching with vector-based semantic similarity, helps improve the relevance of search results. This makes it easier for developers to find the code samples they need.

I hope this article has been helpful in understanding how to set up a simple CSLA MCP server. If you have any questions or need further assistance, feel free to reach out on the CSLA discussion forums or GitHub repository for the csla-mcp project.
