---
{"dg-publish":true,"permalink":"/30-projects/double-simulation-loop/hourly-stats/","created":"2026-03-13T09:21:56.127-04:00","updated":"2026-03-16T13:42:13.911-04:00"}
---

`HourlyStats` is a **per-hour aggregation container** for Prescient’s simulation results.

### Its role

During a single simulated hour, Prescient may run multiple **SCEDs** (security-constrained economic dispatch solves). `HourlyStats`:

- **Collects each SCED’s `OperationsStats`** in `operations_stats` (so you can inspect the individual SCED snapshots later).
- **Builds hour-level metrics** by incrementally incorporating each SCED as it finishes via `incorporate_operations_stats()`.

### What it aggregates

It maintains a mix of:

- **Scalar totals (sums across SCEDs in the hour)**  
    Examples: `fixed_costs`, `variable_costs`, `sced_runtime`, `on_offs`, `quick_start_additional_costs`.
    
- **Scalar averages (average across SCEDs in the hour)**  
    Examples: `total_demand`, `power_generated`, `load_shedding`, `price`, `renewables_used`.
    
- **Keyed totals and keyed averages (dicts keyed by model entities)**  
    Examples:
    
    - keyed sums: `thermal_gen_revenue[g]`, `thermal_uplift[g]`, etc.
    - keyed averages: `observed_thermal_dispatch_levels[g]`, `observed_bus_LMPs[b]`, `observed_flow_levels[l]`, etc.
- **Derived read-only properties** that compute things from the aggregated data:
    
    - `sced_count` (number of SCEDs in the hour)
    - `total_costs = fixed_costs + variable_costs`
    - settlement/payment summaries like `thermal_energy_payments`, `reserve_payments`, etc. (only if `options.compute_market_settlements` is enabled)

### Why it exists

It’s essentially the object that turns “many SCED-level stats” into “one coherent **hourly report**” that downstream reporting/plotting/export can use.