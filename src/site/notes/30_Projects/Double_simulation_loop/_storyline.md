---
{"dg-publish":true,"permalink":"/30-projects/double-simulation-loop/storyline/","created":"2026-03-15T14:05:40.971-04:00","updated":"2026-03-16T13:40:32.073-04:00"}
---

- thermal_generator_prescient_plugin.py is the plugin module Prescient loads.
- Prescient looks for the plugin entry points on that module.
- The module exposes register_plugins = coordinator.register_plugins and get_configuration = coordinator.get_configuration.
- So when Prescient calls the plugin’s register_plugins(...), it is actually calling the coordinator’s method.
- The coordinator then registers many callback functions onto Prescient hook points.
- Prescient stores those callbacks in different places depending on hook type.
- Later, during simulation, Prescient invokes those callbacks at the right times.

