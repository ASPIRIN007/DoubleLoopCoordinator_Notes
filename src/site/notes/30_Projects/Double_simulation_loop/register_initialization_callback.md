---
{"dg-publish":true,"permalink":"/30-projects/double-simulation-loop/register-initialization-callback/"}
---

What this function does is it tells Prescient: “when you reach that initialization stage, call self.initialize_customized_results.”

The key sequence is in [[30_Projects/Double_simulation_loop/prescient-simulator-simulator.py\|prescient-simulator-simulator.py]] In order, Prescient does:

1. `engine.initialize(options)`
2. `time_manager.initialize(options)`
3. `data_manager.initialize(engine, options)`
4. `oracle_manager.initialize(engine, data_manager, options)`
5. `stats_manager.initialize(options)`
6. `reporting_manager.initialize(options, stats_manager)`
7. `invoke_initialization_callbacks(options, self)`

So our plugin callback is called only after those core Prescient objects have already been initialized.

What “vanilla Prescient initialization” does,:

- [[30_Projects/Double_simulation_loop/prescient-simulator-prescient.py\|prescient-simulator-prescient.py]] creates the standard managers:`Engine`, `TimeManager`, `DataManager`, `OracleManager`, `StatsManager`, `ReportingManager`.
- [[30_Projects/Double_simulation_loop/prescient-simulator-time_manager.py\|prescient-simulator-time_manager.py]] sets up the simulation calendar:
  start/stop dates, SCED frequency, RUC frequency, activation timing, current time tracking.
- [[30_Projects/Double_simulation_loop/prescient-simulator-data_manager.py\|prescient-simulator-data_manager.py]] sets up simulation state storage:
  current state, pending/active RUC market info, `extensions`, prior SCED instance.
- [[30_Projects/Double_simulation_loop/prescient-simulator-oracle_manager.py\|prescient-simulator-oracle_manager.py]] stores references to the engine and data manager so it can later create/solve RUC and SCED models.
- [[30_Projects/Double_simulation_loop/prescient-simulator-stats_manager.py\|prescient-simulator-stats_manager.py]] initializes the statistics machinery for operations/hourly/daily/overall stats.

Then, after all that, Prescient calls the registered initialization callbacks.

So if we run vanilla Prescient with no plugin callback, those built-in managers and internal state still get initialized normally. Our callback is only an extra step layered on top.

In our IDAES case, <span style="color:rgb(0, 112, 192)">initialize_customized_results</span> just adds plugin-owned storage under `simulator.data_manager.extensions["customized_results"]`. 

That is why it is low-conflict: it augments Prescient’s initialized state rather than replacing the whole initialization process.