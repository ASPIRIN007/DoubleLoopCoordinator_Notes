---
{"dg-publish":true,"permalink":"/30-projects/double-simulation-loop/placeholder-future-callbacks/","created":"2026-03-14T18:00:30.250-04:00","updated":"2026-03-16T13:42:52.605-04:00"}
---

These also appear in plugin_registration.py, but they are not active entries in the callbacks list because they are not implemented yet.

Examples:

- register_before_ruc_generation_callback
- register_before_new_forecast_ruc_callback
- register_before_new_actuals_ruc_callback
- register_before_ruc_activation_callback
- register_before_operations_callback
- register_dispatch_level_provider_callback
- register_after_lmp_callback

These just raise RuntimeError("This callback is not yet implemented").