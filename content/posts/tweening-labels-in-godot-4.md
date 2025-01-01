---
title: Tweening Labels in Godot 4
date: 2025-01-01
description: "Tweening labels is a fantastic way to add flair to your game’s UI, and adopting a component-based architecture makes it a breeze to reuse these effects across multiple areas of your project without duplicating code."
image: images/posts/im-not-going-to-publish-a-game-this-year-banner.webp
imageAltAttribute: A bad image showcasing some of the label tween effects.
tags:
   - game development
   - godot
---

Tweening labels is a fantastic way to add flair to your game’s UI, and adopting a component-based architecture makes it a breeze to reuse these effects across multiple areas of your project without duplicating code.

While working on [Jurassic Jump](/tags/jurassic-jump), I wanted to spice up my labels with animations. Godot’s tweening system is incredibly versatile, and so far, I’ve implemented two effects: **scrambling and revealing text** and **enlarging text**. Let’s dive into how these work and why they’re so effective.

## Scrambling and Revealing Text 

This is a really cool effect I use in the splash screens. Here's how it looks in action:

![Jurassic Jump Scramble Label](/images/posts/jurassic-jump-scramble-label.gif)

What’s cool is that this effect can double as a text reveal, perfect for secrets, passwords, or suspenseful dialogue reveals:

![Jurassic Jump Reveal Label](/images/posts/jurassic-jump-reveal-label.gif)

The best part? The code is clean and reusable when extracted into its own TweenLabel scene.

```gdscript
## A control for displaying text using various tween effects
class_name TweenLabel extends Label


## Tween text over a period of time, which results in a scramble effect for the
## characters that have changed. Does not scramble characters that are the same
## between calls.
func tween_text(text: String, scramble_time: float, show_time: float) -> Tween:
	var tween = create_tween()
	tween.tween_property(self, "text", text, scramble_time)
	tween.tween_interval(show_time)
	return tween
```

With just a few lines of code, you can create effects that add polish and personality to your game’s UI.

## Enlarging Text 

For countdowns or dramatic emphasis, enlarging labels over time is a great trick. Here’s how it looks in Jurassic Jump’s countdown timer:

![Jurassic Jump Countdown Label](/images/posts/jurassic-jump-countdown-label.gif)

This effect uses a `tween_font_size function`. While I ran into some hiccups with theme compatibility, the results are worth the effort:

```gdscript
## Tween font size over a period of time, which causes a label to grow or shrink
## in size over time. 
func tween_font_size(font_size: int, duration: float) -> Tween:
	var tween := create_tween()
	tween.tween_property(self, "label_settings:font_size", 200, duration)
	return tween
```

The implementation could benefit from further refinement—like creating a dedicated `CountdownLabel` to simplify timing and orchestration — but for now, it works well enough.

## Wrapping Up

Godot’s tweening system opens up endless possibilities for animating labels and other UI elements. Whether it’s scrambling text for a splash screen or enlarging it for dramatic effect, these techniques can help elevate your game’s polish.

I hope this post sparks ideas for your projects (or serves as a handy reference when I inevitably forget how I did this).

Happy tweening! 🚀 
