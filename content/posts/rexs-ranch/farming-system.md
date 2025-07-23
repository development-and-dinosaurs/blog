---
title: Rex's Ranch Farming System
description: The farming system for Rex's Ranch is finally complete! Well... it's "complete". It'll probably never be *finished* finished, but the core gameplay loop around farming is now complete.
date: 2025-04-26
tags:
   - game development
   - godot
   - rex's ranch
---

The farming system for Rex's Ranch is finally complete! Well... it's "complete". It'll probably never be *finished* finished, but the core gameplay loop around farming is now complete.

![Rex's Ranch Screenshot](/images/posts/rexs-ranch/farming-system.webp)

## What's farming like in Rex's Ranch?

Really cool! We've got all of the expected farming actions implemented now — tilling farmland, planting seeds, watering plants, harvesting crops, and shipping produce. Crops grow overnight as long as they've been watered, and take varying amounts of time to reach maturity to be ready for harvest — and they're worth different amounts when shipped too.

## What's different about farming in Rex's Ranch? 

Rex's Ranch is a desktop game — an idle farming simulator that sits on the desktop whilst you do other things. So the first part that's different is the lack of most forms of direct interaction. You don't really "control a player" in this game. The farmer will do things — sometimes things that you've specifically asked to be done like till an area of farmland or plant a specific seed. Other times they'll just do what needs to be done — watering crops, harvesting crops, or taking a little rest.

We don't have money — you're not a farmer that is looking to get rich off the fact that everyone needs to eat. The "currency" we earn in Rextopia is "Reputation" — how well known and well esteemed you are amongst the community. Whether this ends up becoming a thin facade for money I'm not sure yet, but the plan is for no money for a cooperative experience in the community. 

## What's next for Rex's Ranch?

Tying off some of the loose ends in the farming system that I'm saying is complete. Because it's not really, we've just got the barebones implementation in place. There are a couple of features I want to add to it before moving on to the next big system (Which may well end up being fishing — or the much less glamourous debris clearing). 

Some of the things I'm looking at are perennial or regrowable crops — think strawberries, cucumbers, and green beans that can be harvested multiple times within a season without needing to be replanted. The crop unlocking system needs some work too — at the start of the game, most crops are locked with requirements to unlock them. But... doing it doesn't actually unlock them yet, which is unfortunate. 

Alright that's it for now — see you next time when there's hopefully a much smaller update rather than 11,000 lines of farming system.
