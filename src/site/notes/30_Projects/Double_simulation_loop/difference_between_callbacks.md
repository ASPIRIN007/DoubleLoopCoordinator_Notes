---
{"dg-publish":true,"permalink":"/30-projects/double-simulation-loop/difference-between-callbacks/","created":"2026-03-14T17:07:48.359-04:00","updated":"2026-03-16T13:41:53.785-04:00"}
---

>[!Question]
>What does lines context.register_xyz_hooks(abc function) does?
If I understand it correctly it registers the functions into prescient plugin for it to be called back when the time of execution comes. if my understanding is right, where does these function gets registered? are all different functions based on their hooks get's registered in the different places? like register for hourly stats register new subscribers into pending hourly stats subscribers list. Does all the hooks have such pending lists?


Yes, your understanding is right.

A line like:

```python
context.register_xyz_callback(my_function)
```

means:

- “tell Prescient that when hook `xyz` occurs, call `my_function`”

So this is registration, not execution.

Where the function gets registered depends on the kind of hook.

There are two main patterns in Prescient.

**1. [[30_Projects/Double_simulation_loop/Normal lifecycle callbacks\|Normal lifecycle callbacks]]**

**2. [[30_Projects/Double_simulation_loop/Stats subscription hooks\|Stats subscription hooks]]**

**3. [[30_Projects/Double_simulation_loop/Placeholder-future callbacks\|Placeholder-future callbacks]]**  


**Do all hooks have pending lists?**
No.

Only the stats-subscription hooks use the special “pending subscribers” mechanism.

Normal lifecycle hooks do not use pending lists. They are just stored directly in their corresponding callback lists inside `PluginCallbackManager`.

So the distinction is:

- lifecycle hooks -> stored directly in per-hook callback lists
- stats hooks -> first stored in pending subscriber lists, then moved to `stats_manager`

A very compact mental model:

- `register_before_ruc_solve_callback(f)` -> add `f` to the “before RUC solve” callback list
- `register_after_operations_callback(f)` -> add `f` to the “after operations” callback list
- `register_for_hourly_stats(f)` -> add `f` to pending hourly subscribers, later attach to `stats_manager`

So yes:
- different hooks register in different places
- but not every hook has a pending list
- only stats subscriptions do