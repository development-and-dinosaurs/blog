---
title: Using a Finite State Machine for Farmer Behaviours
description: Early on in development I swapped out my finite state machine implementation with a Behaviour Tree for the farmer, but I think there's room for both.
date: 2025-05-03
tags:
   - game development
   - godot
   - rex's ranch
---

Early on in development I swapped out my finite state machine implementation with a behaviour tree for the farmer, but I think there's room for both.

Dont' get me wrong - the behaviour tree is _fantastic_ and it's made the farmer behaviour so much simpler to implement. But by completely getting rid of the concept of the finite state machine, it's made implementing the individual behaviours a lot harder than it needed to be if I had used the best of both.

Anyway here's the previous version of the `HarvestCropTask` in the behaviour tree:

```gdscript
class_name HarvestCropTask extends Task

var can_harvest_crop: bool = true
var crop_harvested: bool = false
var can_place_crop: bool = true
var crop_placed: bool = false
var target_area: Crop

func _init(area: Crop):
	target_area = area

func execute(actor: FarmActor, delta: float) -> int:
	SignalBus.farmer_harvesting.emit()
	if !crop_harvested:
		actor.move_to(target_area.global_position)
		if !actor.navigation_finished():
			return BTNode.BTStatus.RUNNING
	if can_harvest_crop && !crop_harvested:
		can_harvest_crop = false
		crop_harvested = actor.harvest(target_area)
		GlobalTimer.create_timer(.5).timeout.connect(reset_harvest_crop)
	if can_place_crop && crop_harvested:
		actor.carrying = true
		var carry_crop := target_area.harvest()
		if carry_crop == null:
			return BTNode.BTStatus.RUNNING
		actor.add_child(carry_crop)
		carry_crop.position = Vector2(0, -10)
		var shipping_box := actor.get_tree().get_first_node_in_group("shipping_box")
		actor.move_to(shipping_box.global_position)
		if !actor.navigation_finished():
			return BTNode.BTStatus.RUNNING
		crop_placed = actor.ship(shipping_box, carry_crop.data)
		var tween := carry_crop.create_tween()
		tween.tween_property(carry_crop, "global_scale", Vector2(0.1, 0.1), 0.3)
		can_place_crop = false
		GlobalTimer.create_timer(0.8).timeout.connect(reset_place_crop)
		if crop_placed:
			actor.carrying = false
			carry_crop.queue_free()
			return BTNode.BTStatus.SUCCESS
	return BTNode.BTStatus.RUNNING
```

For the sake of my reputation I probably should have cleaned this up a bit first but that's ok - it's messy, it's convoluted, and it is hard to extend. This is partway through adding regrowable crops to the game and struggling to get that implemented. This will make it easier to prove the point that a finite state machine can make all of those problems go away with a little bit of refactoring. 

So the post picture spoils the outcome a little

```gdscript
class_name HarvestCropTask extends Task

enum HarvestState { WAIT, MOVE_TO_CROP, HARVEST, PICKUP, MOVE_TO_BOX, SHIP, DONE }

var state: int = HarvestState.MOVE_TO_CROP
var target_crop: Crop
var shipping_box: ShippingBox
var carried_crop: Crop

func _init(crop: Crop):
	target_crop = crop
	shipping_box = area.get_tree().get_first_node_in_group("shipping_box")

func execute(actor: FarmActor, delta: float) -> int:
	SignalBus.farmer_harvesting.emit()
	match state:
		HarvestState.MOVE_TO_CROP: 
			_move_to_crop(actor)
		HarvestState.HARVEST: 
			_harvest_crop(actor)
		HarvestState.PICKUP: 
			_pick_up_crop(actor)
		HarvestState.MOVE_TO_BOX: 
			_move_to_box(actor)
		HarvestState.SHIP: 
			_ship_crop(actor)
		HarvestState.DONE: 
			return BTNode.BTStatus.SUCCESS
		HarvestState.WAIT: 
			pass
	return BTNode.BTStatus.RUNNING

func _move_to_crop(actor: FarmActor) -> void:
	actor.move_to(target_area.global_position)
	if actor.navigation_finished():
		state = HarvestState.HARVEST

func _harvest_crop(actor: FarmActor) -> void:
	actor.harvest(target_area)
	GlobalTimer.create_timer(0.5).timeout.connect(func(): state = HarvestState.PICKUP)
	state = HarvestState.WAIT

func _pick_up_crop(actor: FarmActor) -> void:
	carried_crop = target_area.harvest()
	actor.carry(carried_crop)
	state = HarvestState.MOVE_TO_BOX

func _move_to_box(actor: FarmActor) -> void:
	actor.move_to(shipping_box.global_position)
	if actor.navigation_finished():
		state = HarvestState.SHIP

func _ship_crop(actor: FarmActor) -> void:
	actor.ship(shipping_box, carried_crop.data)
	var tween := carried_crop.create_tween().parallel()
	tween.tween_property(carried_crop, "global_position",shipping_box.global_position, 0.3)
	tween.tween_property(carried_crop, "global_scale", Vector2(0.1, 0.1), 0.3)
	GlobalTimer.create_timer(0.3).timeout.connect(func(): state = HarvestState.DONE)
	state = HarvestState.WAIT
```

It's a bit longer now - but that's mostly because things are neatly separated to where they need to be rather than just splatted in a single function. We could probably have separated things out with implementing a finite state machine, but the separation into states makes things so much easier to reason about what should happen where.

Before we had a lot of guard statements to ensure that things only happened once or for only a certain amount of time, which is now covered by the state that the task is in. We've still got the concept of waiting for a step to finish, but now it's controlled by the `WAIT` state and transitioning from that to the next state using a timeout.

This state machine is implemented much more simply than the node-based one I was using before - by storing the state within the task script itself and checking it when we execute the task as part of the behaviour tree. This makes it really easy to use in conjunction with the behaviour tree and have everything self-contained within that task node. 

Anyway, regrowable crops now (nearly) works, and I'll look at getting the other tasks migrating over to a state machine architecture where it makes sense. For now, let's go regrowable crops properly implemented and live.
