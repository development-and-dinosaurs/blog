---
title: More State Machines For Rex's Ranch
description: Spending more time changing the state machines in Rex's Ranch for fun and profit. Well... just fun I guess, there's no profit here and there won't be if I keep refactoring my state machines instead of implementing the game.
date: 2025-09-24
tags:
   - game development
   - godot
   - rex's ranch
---

Spending more time changing the state machines in Rex's Ranch for fun and profit. Well... just fun I guess, there's no profit here and there won't be if I keep refactoring my state machines instead of implementing the game.

I thought I was joking in my last "next time" line but apparently not, I am really enjoying state machines right now.

So in the previous example, we moved one of the task classes over to be represented more closely with a standard definition of a state machine - using terms like `State`, `Transition`, and `Event`. However, everything was defined separately in each of the `Task` classes and didn't "have to" follow the same kind of pattern. We were also missing some of the other concepts that maybe we didn't need, but could have made some parts of the state machine easier to implement.

For example - when catching fish we had a couple of "duplicate" states and events that differed in name, as we didn't have the concept of a `Guard` or event parameters to work from. This wasn't the worst thing in the world, but adding these concepts would make the implementation neater.

I also took this chance to move to a more "formal" state machine executor that could parse a definition and do what needed to be done rather than reimplement a whole state machine in each and every task.

This is still a work in progress and doesn't have everything it needs yet to be reusable across everything, but it's getting there. Here's the new `StateMachine` that takes the definition and reacts to events to transition to the next state.

```gdscript
class_name StateMachine extends RefCounted

var _current_state: State
var _event_queue: Array = []
var _states: Dictionary = {}


func _init(definition: Dictionary):
	_load_definition(definition)
	set_initial_state(definition.get("initialState"))


func _load_definition(definition: Dictionary):
	for state_name in definition["states"]:
		var state_obj := State.new(state_name)
		var state_data: Dictionary = definition["states"][state_name]

		state_obj.is_final = state_data.get("is_final", false)
		_states[state_name] = state_obj

	for state_name in definition["states"]:
		var state_data = definition["states"][state_name]
		var state_obj = _states[state_name]

		if state_data.has("on"):
			for event in state_data["on"]:
				var transition_data = state_data["on"][event]
				_load_transitions(state_obj, event, transition_data)


func _load_transitions(state_obj: State, event, transition_data) -> void:
	if transition_data is Array:
		for transition in transition_data:
			var target_state = _states[transition["target"]]
			var action_callable = transition.get("action", Callable())
			var guard_callable = transition.get("guard", Callable())
			state_obj.add_transition(event, target_state, action_callable, guard_callable)
	else:
		var target_state = _states[transition_data["target"]]
		var action_callable = transition_data.get("action", Callable())
		var guard_callable = transition_data.get("guard", Callable())
		state_obj.add_transition(event, target_state, action_callable, guard_callable)


func process_next_event() -> void:
	if not _event_queue.is_empty():
		var event_to_process = _event_queue.pop_front()
		_handle_transition(event_to_process)


func raise_event(event: int) -> void:
	_event_queue.push_back(event)


func get_current_state() -> State:
	return _current_state


func set_initial_state(state_name: String) -> void:
	if _states.has(state_name):
		_current_state = _states[state_name]
	else:
		push_error("StateMachine: Initial state not found: ", state_name)


func _handle_transition(event: int) -> void:
	var transitions = _current_state.transitions.get(event)
	if not transitions:
		return

	if transitions is Array:
		for transition in transitions:
			if not transition.guard.is_valid() or transition.guard.call():
				_execute_transition(transition)
				return
	else:
		if not transition.guard.is_valid() or transition.guard.call():
		 _execute_transition(transitions)


func _execute_transition(transition: Transition) -> void:
	if transition.action.is_valid():
		transition.action.call()
	_current_state = transition.target_state


class State extends RefCounted:

	var name: String
	var transitions: Dictionary = {}
	var is_final: bool = false

	func _init(state_name: String):
		self.name = state_name

	func add_transition(event: int, target_state: State, action: Callable, guard: Callable):
		if transitions.has(event):
			if not transitions[event] is Array:
				transitions[event] = [transitions[event]]
			transitions[event].append(Transition.new(target_state, action, guard))
		else:
			transitions[event] = Transition.new(target_state, action, guard)


class Transition extends RefCounted:

	var target_state: State
	var action: Callable
	var guard: Callable

	func _init(target: State, action_callable: Callable = Callable(), guard_callable: Callable = Callable()):
		self.target_state = target
		self.action = action_callable
		self.guard = guard_callable
```

Ah that's a lot of lines, but it means less lines elsewhere? And a more consistent approach across the codebase.

So we've still got `State`, `Transition`, `Event`, and `Action` that we had before. We've added `Guard` to ensure we pick the correct transition off the back of an event. More importantly, this lets us define our state machine configuration as data that we can just pass into the `StateMachine` class and have it do everything we need.

Guards and actions are still implemented in the tasks which we could improve a little bit - maybe in the future. Here's what a task looks like now, using the `ChopWoodTask` from before to show the difference:

```gdscript
class_name ChopWoodTask extends Task

const NAME := "ChopWoodTask"
const TYPE := "direct"

enum Event {
	TASK_STARTED, ARRIVED_AT_TARGET, DISTANCE_INCREASED, TREE_CHOPPED_DOWN, CHOP_STARTED, CHOP_FINISHED
}

var _fsm: StateMachine = StateMachine.new({
	"initialState": "IDLE",
	"states": {
		"IDLE": {
			"on": {
				Event.TASK_STARTED: {"target": "MOVING", "action": _move},
			}
		},
		"MOVING": {
			"on": {
				Event.ARRIVED_AT_TARGET: {"target": "CHOPPING", "action": _chop},
			}
		},
		"CHOPPING": {
			"on": {
				Event.CHOP_STARTED: {"target": "COOLDOWN"},
				Event.DISTANCE_INCREASED: {"target": "MOVING", "action": _move},
			}
		},
		"COOLDOWN": {
			"on": {
				Event.CHOP_FINISHED: {"target": "CHOPPING", "action": _chop},
				Event.TREE_CHOPPED_DOWN: {"target": "COMPLETE"}
			}
		},
		"COMPLETE": {
			"is_final": true
		}
	}
})

var _target_tree: FarmTree


func _init(tree: FarmTree):
	_target_tree = tree
	SignalBus.tree_chopped.connect(_on_tree_chopped)


func execute(_delta: float) -> int:
	if !is_instance_valid(_target_tree):
		return BTNode.BTStatus.SUCCESS

	_raise_internal_events()
	_fsm.process_next_event()

	if _fsm.get_current_state().is_final:
		return BTNode.BTStatus.SUCCESS

	return BTNode.BTStatus.RUNNING


func _raise_internal_events() -> void:
	match _fsm.get_current_state().name:
		"IDLE":
			_fsm.raise_event(Event.TASK_STARTED)
		"MOVING":
			if actor.navigation_finished():
				_fsm.raise_event(Event.ARRIVED_AT_TARGET)


func _on_tree_chopped(tree: FarmTree) -> void:
	if tree == _target_tree:
		_fsm.raise_event(Event.TREE_CHOPPED_DOWN)


func _move() -> void:
	actor.move_to(_target_tree.global_position)


func _chop() -> void:
	if actor.global_position.distance_squared_to(_target_tree.global_position) > 1000:
		_fsm.raise_event(Event.DISTANCE_INCREASED)
		return
	actor.chop_wood(_target_tree)
	_fsm.raise_event(Event.CHOP_STARTED)
	GlobalTimer.create_timer(1.0).timeout.connect(_fsm.raise_event.bind(Event.CHOP_FINISHED))
```

Uh probably a bad example actually as this doesn't show the new stuff like guards but that's fine - we can see how this state machine definition fits into the `StateMachine` and has the same stuff and the same behaviour it had before, and we can look at guards in more detail later.

Still using an enum for the event because I didn't think of a good way to incorporate those into the state machine definition without then having to reference them by name across different places, but either way it's kind of the same, as some events are outside the realm of the state machine anyway.

So this is working the same way as before in regards to behaviour - when we're in the `IDLE` state, a `TASK_STARTED` event will trigger a transition to the `MOVING` state, whilst in the `MOVING` state we'll be checking where we are and raise the `ARRIVED_AT_TARGET` event when we arrive at the target, which will trigger the `CHOPPING` state, and so on. All of this is controlledvia the data in the state machine, and the events that are raised from actions and the environment of the state machine.

I want to add some more stuff to guards so they don't need to be specific to the task classes any more, so maybe we'll look at this yet again sometime soon.
