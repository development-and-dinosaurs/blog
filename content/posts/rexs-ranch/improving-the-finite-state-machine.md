---
title: Improving the Finite State Machine
description: So I've been working at my actual job that actually brings in money, and we've been looking at using finite state machines for some of the things we're building right now.
date: 2025-09-16
tags:
   - game development
   - godot
   - rex's ranch
---

So I've been working at my actual job that actually brings in money, and we've been looking at using finite state machines for some of the things we're building right now.

Because there's money involved (from them, to me) I feel obliged to do a good job, which means extensive reading into state machines from authoritative sources who know a lot more about it than I do, rather than just random blog posts on the internet from people with questionable credentials (Hi by the way 👋).

The upside is that now I am a better employee more capable of doing my work. But more importantly I also know a bunch more about state machines that I can apply to my game as another refactoring effort rather than actually progressing with any of my features.

Let's take a look at the changes and why I think this implementation is better than before. First, this is what I had before:

```gdscript
class_name ChopWoodTask extends Task

static var NAME = "ChopWoodTask"

enum State { READY, MOVING, CHOPPING, COOLDOWN, COMPLETE }

var state: State = State.READY
var target: FarmTree


func _init(tree: FarmTree):
	target = tree


func execute(actor: FarmActor, _delta: float) -> int:
	if !is_instance_valid(target):
		return BTNode.BTStatus.SUCCESS

	match state:
		State.READY:
			actor.move_to(target.global_position)
			state = State.MOVING

		State.MOVING:
			if actor.navigation_finished():
				state = State.CHOPPING
			return BTNode.BTStatus.RUNNING

		State.CHOPPING:
			if actor.global_position.distance_squared_to(target.global_position) > 1000:
				state = State.READY
				return BTNode.BTStatus.RUNNING
			chop(actor)
			return BTNode.BTStatus.RUNNING

		State.COOLDOWN:
			return BTNode.BTStatus.RUNNING

		State.COMPLETE:
			return BTNode.BTStatus.SUCCESS

	return BTNode.BTStatus.RUNNING


func chop(actor: FarmActor) -> void:
	var finished := actor.chop_wood(target)
	if finished:
		state = State.COMPLETE
	else:
		state = State.COOLDOWN
		GlobalTimer.create_timer(1.0).timeout.connect(func(): state = State.CHOPPING)
```

Now I don't think this looks _too_ bad at first glance. The farmer can be in one state at a time. Being in a state is tied to the stuff the farmer does in that state - chopping, moving, waiting, etc. And we can transition from one state to another based on things that have happened.

The main problem I've got is that a bunch of the stuff is implicit rather than explicit, and we're not really leaning on the functions and constraints of a well-defined finite state machine, which would help with understanding how it works at a glance, and communicating that with others where required.

So to make things more explicit, we want to lean on some of the terminology, constraints, and features of a finite state machine. The things we'll need to have to do this are:

- States. We've got these already, so that should be fine.
- Transitions. We transition between states but this is all implicit, but it shouldn't be too bad to formalise this in the structure.
- Events. We don't _really_ have events in the current system. We want to be able to raise internal events from the actions we're taking in the state machine, and also react to events that happen outside.
- Actions. We have some actions that the farmer does like move and chop, but they're inside the states implicitly rather than something that we call out explicitly.

The way we want this task to work is: we're in a state, we react to events, events trigger a transition into a different state, and we execute actions as we're transitioning between states. So this is what we've got now:

```gdscript
class_name ChopWoodTask extends Task

static var NAME = "ChopWoodTask"

enum State { IDLE, MOVING, CHOPPING, COOLDOWN, COMPLETE }
enum Event { TASK_STARTED, ARRIVED_AT_TARGET, DISTANCE_INCREASED, TREE_CHOPPED_DOWN, CHOPPED, CHOP_DONE }

var _transitions: Dictionary = {
	State.IDLE: {
		Event.TASK_STARTED: { "action": _start_moving, "target": State.MOVING },
	},
	State.MOVING: {
		Event.ARRIVED_AT_TARGET: { "action": _start_chopping, "target": State.CHOPPING }
	},
	State.CHOPPING: {
		Event.DISTANCE_INCREASED: { "target": State.IDLE },
		Event.CHOPPED: { "target": State.COOLDOWN },
	},
	State.COOLDOWN: {
		Event.CHOP_DONE: { "action": _start_chopping, "target": State.CHOPPING },
		Event.TREE_CHOPPED_DOWN: { "target": State.COMPLETE }
	}
}
var _event_queue: Array = []
var _state: State = State.IDLE
var _target_tree: FarmTree


func _init(tree: FarmTree):
	_target_tree = tree
	SignalBus.tree_chopped.connect(_on_tree_chopped)


func execute(actor: FarmActor, _delta: float) -> int:
	if !is_instance_valid(_target_tree):
		return BTNode.BTStatus.SUCCESS

	_generate_internal_events(actor)
	_handle_events(actor)

	match _state:
		State.COMPLETE:
			return BTNode.BTStatus.SUCCESS
		_:
			return BTNode.BTStatus.RUNNING


func _generate_internal_events(actor: FarmActor) -> void:
	match _state:
		State.IDLE:
			_raise_event(Event.TASK_STARTED)
		State.MOVING:
			if actor.navigation_finished():
				_raise_event(Event.ARRIVED_AT_TARGET)
		State.CHOPPING:
			if actor.global_position.distance_squared_to(_target_tree.global_position) > 1000:
				_raise_event(Event.DISTANCE_INCREASED)


func _handle_events(actor: FarmActor) -> void:
	while !_event_queue.is_empty():
		var event_to_process = _event_queue.pop_front()
		_handle_transition(actor, event_to_process)


func _handle_transition(actor: FarmActor, event: Event) -> void:
	var transition_data = transitions.get(_state, {}).get(event)

	if transition_data:
		var action_function = transition_data.get("action")
		var target = transition_data.get("target")

		if action_function:
			action_function.call(actor)

		if target != null:
			_state = target


func _on_tree_chopped(tree: FarmTree) -> void:
	if tree == _target_tree:
		_raise_event(Event.TREE_CHOPPED_DOWN)


func _raise_event(event: Event) -> void:
	_event_queue.push_back(event)


func _start_moving(actor: FarmActor) -> void:
	actor.move_to(_target_tree.global_position)


func _start_chopping(actor: FarmActor) -> void:
	actor.chop_wood(_target_tree)
	_raise_event(Event.CHOPPED)
	GlobalTimer.create_timer(1.0).timeout.connect(_raise_event.bind(Event.CHOP_DONE))
```

It's longer again but that's because it's more explicit, and now the stuff that we'd need to keep in our heads is in the script instead, which is a better place for it. Relatedly, the events we care about and the way we move from one state to another are really clearly defined rather than within the structure of the state machine itself.

Another benefit is that it's really easy to respond to events that happen outside of the task state machine. Before we were relying on the farmer telling us when the tree is chopped down, which was always a bit weird. Now we can just listen for the SignalBus telling us the tree has been chopped down, which we can add to our event queue to handle it and transition. This is much cooler!

Lastly, it should be easier to talk about with others in familiar terms. State machines work in a particular way, so now when we say "when we receive a tree chopped event, we transition from the cooldown state to the complete state which ends the execution", that will make sense to anyone that's familiar with state machines and that terminology.

Join me next time when we'll reimplement a state machine again to be marginally better than it was before.
