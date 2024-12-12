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

### Solution

The solution (thanks to [@jeffhandley](https://elk.zone/fosstodon.org/@jeffhandley@hachyderm.io)) is to add `type="button"` to all non-submit `button` components.

It turns out that the default HTML for `<button />` is `type="submit"`, so if you don't override that value, then all buttons trigger a submit.

What this means is that I _could_ shorten my actual submit button:

```html
<button class="btn btn-primary">Save</button>
```

I probably won't do this though, as being explicit probably increases readability.

And I _absolutely must_ be explicit with all my other buttons:

```html
<button type="button" class="btn btn-secondary" @onclick="CancelChoice">Cancel</button>
```

This prevents the other buttons (even in nested Razor components) from accidentally triggering the submit behavior in the `EditForm` component.