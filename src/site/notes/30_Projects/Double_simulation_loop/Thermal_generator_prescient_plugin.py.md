---
{"dg-publish":true,"permalink":"/30-projects/double-simulation-loop/thermal-generator-prescient-plugin-py/"}
---


### The "Plugin Connection" Conversation

1. **Prescient:** "I've been told to load the module `thermal_generator_plugin`. Let me look inside."
2. **Prescient:** "Hey, `thermal_generator_plugin`, do you have a `register_plugins` function? I need to know what you want to do during my simulation."
3. **thermal_generator_plugin.py:** "I don't have my own unique code for that, but I've pointed my `register_plugins` label to the `coordinator` object I just built. Go talk to him!"
4. **Prescient:** (Follows the pointer to `coordinator.py`) "Okay, Coordinator, I see your `register_plugins` method. What's the plan?"
5. **coordinator.py:** "Glad you asked. Every time you are about to solve a RUC, call my `bid_into_DAM` function. Every time you finish an hour, call my `push_hourly_stats` function..."
6. **Prescient:** "Got it. I've added all those functions to my 'To-Do' list. We are now connected."

---

### Why this "Pointer" (the `=` sign) is so powerful

In your [thermal_generator_plugin.py](https://github.com/IDAES/idaes-pse/commit/e7e13b89ed22d3d0c33ce9c902a35bff5a947558) file, this specific line is the bridge:

`register_plugins = coordinator.register_plugins`

- **Before this line:** The functions like `bid_into_DAM` are just "dormant" code sitting inside `coordinator.py`. They aren't doing anything.
- **After this line:** You have officially "exposed" those functions to Prescient.
- **The Result:** You created a specific **instance** of a coordinator (configured for a thermal generator) and told Prescient to use that specific instance's logic.

### Summary of the "Chain of Command"

1. **Prescient** (The Engine)
2. $\rightarrow$ calls **`thermal_generator_plugin.py`** (The Entry Point)
3. $\rightarrow$ which redirects to **`coordinator.py`** (The Logic Provider)
4. $\rightarrow$ which registers **Callbacks** (The individual functions like `bid_into_DAM`).

---
