---
title: Introducing Rex
description: Rex's Ranch is a dinosaur-themed farming simulator, but up until now there weren't really any... well... dinosaurs. That has been remediated. Meet Rex!
date: 2025-04-27
tags:
   - game development
   - godot
   - rex's ranch
---

Rex's Ranch is a dinosaur-themed farming simulator, but up until now there weren't really any... well... dinosaurs. That has been remediated. Meet Rex!

![Rex's Ranch Screenshot](/images/posts/rexs-ranch/walk.gif)
![Rex's Ranch Screenshot](/images/posts/rexs-ranch/till.gif)
![Rex's Ranch Screenshot](/images/posts/rexs-ranch/spread.gif)
![Rex's Ranch Screenshot](/images/posts/rexs-ranch/water.gif)
![Rex's Ranch Screenshot](/images/posts/rexs-ranch/harvest.gif)
![Rex's Ranch Screenshot](/images/posts/rexs-ranch/carry.gif)
![Rex's Ranch Screenshot](/images/posts/rexs-ranch/place.gif)
![Rex's Ranch Screenshot](/images/posts/rexs-ranch/sit.gif)

Rex is the farmer in Rex's Ranch, and he's really happy to meet you! These animations are the first pass drafts which will probably get a little bit of cleaning up in the future, but I think they're really cool so far! 

This is the first real piece of artwork I've done — probably ever. The majority of the art in Rex's Ranch is from texture packs, but dinosaur-themed texture packs are few and far between — so I had to do it myself. And actually, I think I've done a really good job of it, if I do say so myself.

I realise at this point I've resigned myself to having to create a new spritesheet each time I add a new action — which is going to take longer that not doing that. But actually it's not too bad now the original design is done — it only took around a day to get all of these animations completed, which included taking some time out to watch Jurassic Park (for inspiration I promise, I wasn't slacking). So, adding a new animation shouldn't take a massive amount of time. 

The only existing animation that hasn't been recreated yet is chopping wood, because I broke the wood chopping system at some point when working on the farming system. But I'm working in priority order, so that's further down the list to (re)implement.

Anyway that's all for now — next up I'm going to be looking at modifying the navigation script so that we hone in on the tile to the left or right of the target tile, so the animations look like they're actually applying to the tile we want. This should make everything look a bit more polished which will be cool. Might need to focus on the center of the tiles too so we're not offset — we'll see what it looks like. 
