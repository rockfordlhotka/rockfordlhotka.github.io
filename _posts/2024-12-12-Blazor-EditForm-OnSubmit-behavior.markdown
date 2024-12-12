---
layout: post
title: Blazor EditForm OnSubmit behavior
date: 2024-12-12T00:00:00.0000000-06:00
categories: []
tags: []
published: true
permalink: 
featuredImageUrl: 
---
I am working on the open-source [KidsIdKit](https://github.com/missingchildrenmn/KidsIdKit) app and have encountered some "interesting" behavior with the `EditForm` component and how buttons trigger the `OnSubmit` event.

An `EditForm` is declared similar to this:

```html
    <EditForm Model="CurrentChild" OnSubmit="SaveData">
```

I would expect that any `button` component with `type="submit"` would trigger the `OnSubmit` handler.

```html
<button class="btn btn-primary" type="submit">Save</button>
```

I would also expect that any `button` component _without_ `type="submit"` would _not_ trigger the `OnSubmit` handler.

```html
<button class="btn btn-secondary" @onclick="CancelChoice">Cancel</button>
```

I'd think this was true _especially_ if that second button was in a nested component, so it isn't even in the `EditForm` directly, but is actually in its own component, and it uses an `EventCallback` to tell the parent component that the button was clicked.

### Actual Results

In Blazor 8 I see different behaviors between MAUI Hybrid and Blazor WebAssembly hosts.

In a Blazor WebAssembly (web) scenario, my expectations are met. The secondary button in the sub-component does _not_ cause `EditForm` to submit.

In a MAUI Hybrid scenario however, the secondary button in the sub-component _does_ cause `EditForm` to submit.

I also tried this using the new Blazor 9 MAUI Hybrid plus Web template - though in this case the web version is Blazor server.

In my Blazor 9 scenarios, in _both_ hosting cases the secondary button triggers the submit of the `EditForm` - even though the secondary button is in a sub-component (its own `.razor` file)!

What I'm getting out of this is that we must assume that _any button_, even if it is in a nested component, will trigger the `OnSubmit` event of an `EditForm`.

Nasty!

### Workaround

My current thought in terms of a workaround is to do something like this in my code:

```c#
    private string messageText = string.Empty;
    private bool ignoreSave;

    private void SaveData()
    {
        if (ignoreSave)
        {
            messageText += "|Ignored|";
            ignoreSave = false;
            return;
        }
        messageText += "|Data saved|";
    }

    private void SubComponentComplete()
    {
        messageText += "|SubComponentComplete|";
        ignoreSave = true;
    }
```

The `EditForm` component's `OnSubmit` event is still handled by the `SaveData` method.

The `EventCallback` from the nested component is handled by the `SubComponentComplete` method.

The `ignoreSave` field is used to prevent the erroneous `EditForm` submit from actually doing anything.

Fortunately when a button in the nested component is clicked, the _first_ thing that happens is that the `EventCallback` is invoked first, before the submit. This allows me to do what I originally wanted to do, _plus_ I can set `ignoreSave` to prevent the submit operation (which I don't want) from actually doing anything.

### Thoughts

Is this a bug in the Blazor `EditForm` component? Or is this just the way an html `Form` is supposed to work, where _all_ button elements in a `Form` trigger the submit behavior?

Is this a known issue that I'm just late to discover, or is this something new and unique?

Mysteries of the universe...