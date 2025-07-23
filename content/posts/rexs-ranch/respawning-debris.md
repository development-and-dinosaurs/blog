---
title: Respawning Debris
description: Debris now comes back overnight, which will be a good thing once you can get materials from it, but for now is just a minor annoyance.
date: 2025-05-27
tags:
   - game development
   - godot
   - rex's ranch
---

Debris now comes back overnight, which will be a good thing once you can get materials from it, but for now is just a minor annoyance.

![Rex's Ranch Screenshot](/images/posts/rexs-ranch/respawning-debris.png)

Originally once you'd gotten rid of the inital debris, it'd be gone forever which is fine for keeping things neat, but it also reduces the things to do (is cleaning up the farm a thing to do? I guess it is...) and would stop players from being able to collect materials required for upgrades... once materials and upgrades are in the game.

Debris is mostly non-destructive - debris won't spawn on top of existing debris, paths, the house, the river, or planted crops. That sounds obvious I guess but it uh... didn't do that originally. However debris will spawn on top of tilled ground. It doesn't remove the tilled ground (maybe it should) so it's easy enough to clear.

So trees should probably spawn as seeds or whatever but we're not looking at realism right now, so this is good enough.

Until next time!
