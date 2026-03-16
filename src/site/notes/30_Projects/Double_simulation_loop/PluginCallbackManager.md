---
{"dg-publish":true,"permalink":"/30-projects/double-simulation-loop/plugin-callback-manager/","created":"2026-03-14T17:46:16.951-04:00","updated":"2026-03-16T13:42:59.634-04:00"}
---

### __ init __
inside init we see the line 

self._pending_stats_subscribers = _StatisticsSubscribers()

this lets us store the pending subscribers stats in the [[30_Projects/Double_simulation_loop/StatisticsSubscribers class\|StatisticsSubscribers class]]


## setup_callback

It is creating the callback infrastructure, not the actual callbacks for vanilla Prescient.

More precisely, _setup_callback(...) creates empty containers and helper methods so Prescient is ready to accept callbacks if any plugin registers them.

So after _setup_callback(...), Prescient has things like:

- an empty list for before_ruc_solve callbacks
- a method to register into that list
- a method to invoke everything in that list

But if there are no external plugins, those lists just stay empty.

- _setup_callback(...) = “set up empty hook sockets”
- plugin registration = “plug functions into those sockets”
- runtime invocation = “call whatever is plugged in”

>[!Note]
>You will see a lot of getattr used in the code, why is it used?

`getattr(self, list_name).append(callback)`

- **The Problem:** Prescient has many lists: `_before_ruc_solve_callbacks`, `_after_operations_callbacks`, etc.
- **The Dynamic Solution:** Instead of writing `if cb == "before_ruc_solve": self._before_ruc_solve_callbacks.append(...)`, which would be hundreds of lines of code, it uses `getattr`.
- **How it works:** It turns the string (the `list_name`) into a command. Referring to [[30_Projects/Double_simulation_loop/analogy_conference\|analogy_conference]] It's like saying: _"Go find the drawer labeled with whatever name is currently stored in the `list_name` variable and put this student inside it."_


## registration functions and invoke functions

The registration methods and invocation methods are the two basic halves of the callback mechanism.

**Registration methods**
Examples:
- `register_before_ruc_solve_callback`
- `register_after_ruc_generation_callback`
- `register_after_operations_callback`

What they do:
- take a function as input
- append that function to the corresponding callback list

So for a hook like `before_ruc_solve`, registration is basically:

```python
self._before_ruc_solve_callbacks.append(callback)
```

Meaning:
- “store this function so Prescient can call it later”

**Invocation methods**
Examples:
- `invoke_before_ruc_solve_callbacks`
- `invoke_after_ruc_generation_callbacks`
- `invoke_after_operations_callbacks`

What they do:
- loop over the stored list
- call each function with the runtime arguments for that hook

So conceptually:

```python
for cb in self._before_ruc_solve_callbacks:
    cb(*args, **kwargs)
```

Meaning:
- “the time for this hook has arrived, so call every registered function for it”

So the whole mechanism is:

1. registration method stores the callback
2. simulation reaches that stage
3. invocation method calls all stored callbacks

That is why they look simple. They are simple by design.

The important thing is not that each one contains complex logic, but that together they create the plugin-hook system.

A compact interpretation:

- `register_*` = save function for later
- `invoke_*` = run saved functions now

Why this matters:
Prescient code elsewhere does not need to know which plugin functions exist. It only needs to say things like:

```python
callback_manager.invoke_before_ruc_solve_callbacks(options, simulator, model, date, hour)
```

and the callback manager takes care of calling all registered plugin functions for that stage.

So yes, the registration side is mostly appending to lists, and the invocation side is mostly iterating through those lists and calling the functions. 