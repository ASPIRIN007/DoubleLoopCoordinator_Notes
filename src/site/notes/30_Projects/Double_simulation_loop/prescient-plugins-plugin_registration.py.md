---
{"dg-publish":true,"permalink":"/30-projects/double-simulation-loop/prescient-plugins-plugin-registration-py/"}
---


For the vanilla Prescient flow installed package, ordered by when they matter in runtime, it looks like this:

**Setup Before Simulation Starts**

1. [[30_Projects/Double_simulation_loop/register_plugin\|register_plugin]]
- 

2. `register_options_preview_callback`
- 

3. `register_initialization_callback`
- 

4. `register_for_operations_stats`
- 

5. `register_for_hourly_stats`
- 
5. `register_for_daily_stats`
- 

7. `register_for_overall_stats`
- 

**Initial RUC / Planning Preparation**

8. `register_before_ruc_generation_callback`
- Placeholder only in this package, not implemented.

9. `register_before_new_forecast_ruc_callback`
- Placeholder only, not implemented.

10. `register_before_new_actuals_ruc_callback`
- Placeholder only, not implemented.

11. `register_after_get_initial_actuals_model_for_simulation_actuals_callback`
- 

12. `register_after_get_initial_forecast_model_for_ruc_callback`
- 
13. `register_before_ruc_solve_callback`
- 

14. `register_after_ruc_generation_callback`
- 

**RUC Activation**

15. `register_before_ruc_activation_callback`
- 

16. `register_after_ruc_activation_callback`
- 

**Operations / SCED Stage**

17. `register_after_get_initial_actuals_model_for_sced_callback`
- 

18. `register_before_operations_solve_callback`
- 
17. `register_after_operations_callback`
- 

20. `register_update_operations_stats_callback`
- 

21. `register_for_operations_stats`
- 

**Published Statistics**

22. `register_for_hourly_stats`
- 
22. `register_for_daily_stats`
- 

24. `register_for_overall_stats`
- 

**End of Simulation**

25. `register_finalization_callback`
- 

A compact “timeline view” is:

1. `register_plugin`
2. `options_preview`
3. core Prescient initialization
4. `initialization`
5. stats subscribers attached to `stats_manager`
6. simulation-actuals model hook
7. forecast-RUC model hook
8. `before_ruc_solve`
9. `after_ruc_generation`
10. `after_ruc_activation`
11. SCED initial-actuals-model hook
12. `before_operations_solve`
13. `after_operations`
14. `update_operations_stats`
15. publish operations stats
16. publish hourly stats when hour ends
17. publish daily stats when day ends
18. publish overall stats when simulation ends
19. `finalization`

And the three `before_*` RUC placeholder callbacks plus `before_ruc_activation` are not active in this vanilla package because they just raise “not yet implemented”.

If you want, I can turn this into a two-column table:
- `register_* API name`
- `actual runtime moment when it fires`