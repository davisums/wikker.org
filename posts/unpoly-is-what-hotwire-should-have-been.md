---
title: Unpoly is what Hotwire should have been
description: Hotwire is a good start, but Unpoly shows how much more is possible with progressive enhancement frameworks.
date: 2024-05-09
scheduled: 2024-05-09
tags: startups
layout: layouts/post.njk
image: https://cdn.pixabay.com/photo/2020/08/30/20/54/rice-field-5530707_1280.jpg
---

[HotWire](https://hotwired.dev/) doesn’t feel very Rails-y. It's a step forward from creating SPAs for the tinyest website, but it could have been so much better. It could’ve been like Unpoly.

The problem with Hotwire, is that it's a lot of work. You need WebSockets (for Turbo Streams) as soon as you try to do something slightly complicated. And you’ll need StimulusJS to handle the slightest interactivity.

A lot of boilerplate, very little magic.

## A Hotwire example

Let’s take a pretty common use case in most applications. Showing a form in a modal. Here’s what you’d need to do to create that with HotWire (taken from [this HotWire tutorial](https://www.bearer.com/blog/how-to-build-modals-with-hotwire-turbo-frames-stimulusjs)):

1. Include the modal container somewhere in your page by default, so that you can target it
2. Call the server to GET the form template and target the modal
3. Create  StimulusJS controller to add a close/cancel button to your modal (by removing the content from the DOM)
4. Use Turbo Streams to update your original page (the one underneath the modal) without a page refresh when a post is updated or created
5. Create another SimulusJS controller that listens for server events and closes the modal when a submission is successful

That’s a lot of work for something that feels like it should be relatively simple. 

## How Unpoly does it

Here’s what the same thing would look like with [Unpoly](https://unpoly.com/):

```
<div
  up-href="/post/new"
  up-layer="new"
  up-mode="modal"
  up-target="#post_content"
  up-on-accepted="up.reload('#post_list')"
>
  Add post
</div>
```

This does pretty much the same thing as the HotWire equivalent with almost no Javascript.

My favorite part is the `up-on-accepted` attribute. The `up.reload('#post_list')` Javascript snippet automatically refreshes that part of the page, and in most cases you don’t even need to specify from which URL it should get the latest version.

*This* feels like magic.

Another cool thing is that you can specify multiple targets in the `up-target` attribute, no need for streams or WebSockets!

Or you can use `up-hungry` to automatically update DOM elements whenever they’re included in a response. You can `up-autosubmit` forms or `up-validate` them to let the server know the data shouldn’t yet be persisted. There’s all the [most common layer modes](https://unpoly.com/layer-terminology#available-modes) available by default. An `.up-loading` class is added to elements while they’re being loaded, so you can customize what they look like in that state. `up-instant` and `up-preload` can make clicks feel instant. And [much more](https://unpoly.com/).

And here’s the kicker: Unpoly has been around since 2015.

I've been using Unpoly for the last 6 months to build [VoteJar](https://votejar.io/) and I'm convinced it would have taken me significantly longer to do the same thing with Hotwire. Probably twice as long with an SPA approach. I highly recommend trying [Unpoly](https://unpoly.com/) out.