---
{"dg-publish":true,"permalink":"/30-projects/double-simulation-loop/bid-into-dam/","created":"2026-03-15T14:06:34.613-04:00","updated":"2026-03-16T13:41:06.752-04:00"}
---

## code



This function uses the bidding objects to bid into the DAM
	# check if it is first day
is_first_day = simulator.[[30_Projects/Double_simulation_loop/prescient-simulator-time_manager.py\|prescient-simulator-time_manager.py]].current_time is None
if not is_first_day:
	# solve rolling horizon to get the trajectory
full_projected_trajectory = self.[[30_Projects/Double_simulation_loop/project_tracking_trajectory\|project_tracking_trajectory]](options, simulator, options.ruc_execution_hour)
	# update the bidding model
self.bidder.[[30_Projects/Double_simulation_loop/update_day_ahead_model\|update_day_ahead_model]](`**full_projected_trajectory`)
	# generate bids
bids = self.bidder.[[30_Projects/Double_simulation_loop/compute_day_ahead_bids\|compute_day_ahead_bids]](date=ruc_date)
if is_first_day:
self.current_bids = bids
self.next_bids = bids
	# pass to prescient
self.[[30_Projects/Double_simulation_loop/_pass_DA_bid_to_prescient\|_pass_DA_bid_to_prescient]](options, ruc_instance, bids)

## Walkthrough explanation

**`bid_into_DAM` Walkthrough**

This is the full control-flow path for how the IDAES thermal generator plugin bids into Prescient’s day-ahead market.

---

**1. Plugin Registration**

Prescient first loads the plugin module:
[`thermal_generator_prescient_plugin.py`](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/examples/thermal_generator_prescient_plugin.py)

That module exposes:

- `get_configuration = coordinator.get_configuration`
- `register_plugins = coordinator.register_plugins`

So when Prescient asks the plugin to register itself, it is really calling:
[`coordinator.py`](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/coordinator.py#L58)

Inside `register_plugins(...)`, this line is the key one for DAM bidding:

```python
context.register_before_ruc_solve_callback(self.bid_into_DAM)
```

Source:
[`coordinator.py`](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/coordinator.py#L86)

This means:
- before Prescient solves the RUC, call `bid_into_DAM(...)`

---

**2. Prescient Stores the Callback**

`context.register_before_ruc_solve_callback(...)` is a method on:
[`plugin_registration.py`](/Users/vardhans/Library/Python/3.11/lib/python/site-packages/prescient/plugins/plugin_registration.py)

That forwards registration to the callback manager in:
[`internal.py`](/Users/vardhans/Library/Python/3.11/lib/python/site-packages/prescient/plugins/internal.py)

So `self.bid_into_DAM` gets stored in the callback list for the `before_ruc_solve` hook.

---

**3. Prescient Reaches the RUC Solve Stage**

During simulation, Prescient’s `OracleManager` creates the deterministic RUC model and then invokes the `before_ruc_solve` callbacks.

This happens here:
[`oracle_manager.py`](/Users/vardhans/Library/Python/3.11/lib/python/site-packages/prescient/simulator/oracle_manager.py#L144)

The important line is:
[`oracle_manager.py`](/Users/vardhans/Library/Python/3.11/lib/python/site-packages/prescient/simulator/oracle_manager.py#L156)

So Prescient calls:

```python
bid_into_DAM(options, simulator, ruc_instance, ruc_date, ruc_hour)
```

---

**4. `bid_into_DAM(...)` Starts**

Main function:
[`coordinator.py`](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/coordinator.py#L468)

It receives:

- `options`
- `simulator`
- `ruc_instance`
- `ruc_date`
- `ruc_hour`

Its job is:
- make sure the bidder’s internal model is up to date
- compute DA bids
- write those bids into Prescient’s RUC model before the RUC solve

---

**5. First-Day Check**

Inside `bid_into_DAM(...)`:

```python
is_first_day = simulator.time_manager.current_time is None
```

Source:
[`coordinator.py`](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/coordinator.py#L488)

If it is the first day:
- there is no past operating history to project from

If it is not the first day:
- the bidder model must be advanced so it reflects the unit’s ongoing state

---

**6. Project the Remaining-Day Tracking Trajectory**

On non-first days, `bid_into_DAM(...)` calls:

```python
full_projected_trajectory = self.project_tracking_trajectory(
    options, simulator, options.ruc_execution_hour
)
```

Source:
[`coordinator.py`](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/coordinator.py#L494)

That function is here:
[`coordinator.py`](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/coordinator.py#L321)

What it does:

- reads current Prescient date/hour
- clones the current tracker state into `projection_tracker`
- loops from `ruc_execution_hour` to hour `23`
- assembles projected tracking signals for each hour
- solves the projection tracker
- merges actual daily tracker stats so far with projected stats for the rest of the day

Supporting functions involved:

- `_clone_tracking_model(...)`
  [`coordinator.py`](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/coordinator.py#L366)
- `assemble_project_tracking_signal(...)`
  [`coordinator.py`](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/coordinator.py#L296)

Purpose:
- estimate the unit’s future state/trajectory so the DA bidder starts from a state consistent with current RT/tracking operation

---

**7. Update the Bidder’s Day-Ahead Model**

After projection, `bid_into_DAM(...)` updates the bidder:

```python
self.bidder.update_day_ahead_model(**full_projected_trajectory)
```

Source:
[`coordinator.py`](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/coordinator.py#L498)

This enters:
[`bidder.py`](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/bidder.py#L649)

which calls:
[`bidder.py`](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/bidder.py#L675)

Inside `_update_model(...)`, for every scenario block:

```python
self.bidding_model_object.update_model(b=model.fs[i], **kwargs)
```

Source:
[`bidder.py`](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/bidder.py#L690)

For the thermal generator example, that concrete method is:
[`thermal_generator.py`](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/examples/thermal_generator.py#L545)

It updates:
- unit up/down time state
- last implemented power
- on/off status

through:
- `_update_UT_DT(...)`
- `_update_power(...)`

So this step advances each scenario’s bidding model to the correct current operating state.

---

**8. Compute the Day-Ahead Bids**

Then `bid_into_DAM(...)` does:

```python
bids = self.bidder.compute_day_ahead_bids(date=ruc_date)
```

Source:
[`coordinator.py`](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/coordinator.py#L501)

This enters:
[`bidder.py`](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/bidder.py#L508)

Inside `compute_day_ahead_bids(...)`, the bidder first gets forecast price scenarios:

```python
self.forecaster.forecast_day_ahead_and_real_time_prices(...)
```

Source:
[`bidder.py`](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/bidder.py#L523)

For the thermal generator example, the forecaster object was created as:
[`thermal_generator_prescient_plugin.py`](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/examples/thermal_generator_prescient_plugin.py#L39)

So the actual class used is:
`PlaceHolderForecaster`

defined in:
[`forecaster.py`](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/forecaster.py#L135)

`PlaceHolderForecaster`:
- uses fixed hourly mean/std inputs
- samples DA and RT price scenarios
- returns those forecast price trajectories to the bidder

---

**9. Solve the Bidding Problem**

`compute_day_ahead_bids(...)` then calls:

```python
self._compute_bids(...)
```

Source:
[`bidder.py`](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/bidder.py#L534)

Main routine:
[`bidder.py`](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/bidder.py#L456)

`_compute_bids(...)` does four things:

1. `_pass_price_forecasts(...)`
   - writes the forecast DA/RT prices into each scenario block’s model parameters
   - source:
     [`bidder.py`](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/bidder.py#L724)

2. `self.solver.solve(model, tee=True)`
   - solves the stochastic bidding optimization

3. `_assemble_bids(...)`
   - extracts the bid structure from the solved model

4. `record_bids(...)`
   - records the bid outputs and scenario details

So this is the stage where forecast prices become optimized DA bids.

---

**10. Store Current and Next Bids**

Back in `bid_into_DAM(...)`, the returned bids are stored:

```python
if is_first_day:
    self.current_bids = bids
self.next_bids = bids
```

Source:
[`coordinator.py`](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/coordinator.py#L503)

So the coordinator keeps track of the active and upcoming bids.

---

**11. Pass the DA Bids into Prescient**

Now the key Prescient integration step:

```python
self._pass_DA_bid_to_prescient(options, ruc_instance, bids)
```

Source:
[`coordinator.py`](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/coordinator.py#L508)

Function:
[`coordinator.py`](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/coordinator.py#L272)

What it does:

1. gets the controlled generator name
2. fetches that generator’s data dictionary from the Prescient RUC instance:

```python
gen_dict = ruc_instance.data["elements"]["generator"][gen_name]
```

Source:
[`coordinator.py`](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/coordinator.py#L289)

3. calls `_update_bids(...)` to overwrite Prescient’s generator fields with the bidder’s DA bid data

Supporting function:
[`coordinator.py`](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/coordinator.py#L185)

That function updates fields like:
- `p_cost`
- `p_max`
- `p_min`
- `fixed_commitment`
- `startup_capacity`
- `shutdown_capacity`
- `startup_cost`
- `min_up_time`
- `min_down_time`

So the DA bids are passed into Prescient by directly modifying the one target generator’s in-memory RUC data.

---

**12. Prescient Solves the Modified RUC**

After `bid_into_DAM(...)` returns, control goes back to Prescient’s `OracleManager`.

Now the RUC instance has your generator’s updated bid data, so when Prescient solves the deterministic RUC and then the day-ahead market, it uses your modified bid inputs.

That solve path is in:
[`oracle_manager.py`](/Users/vardhans/Library/Python/3.11/lib/python/site-packages/prescient/simulator/oracle_manager.py#L160)

So the plugin’s DA bid affects the actual Prescient market clearing.

---

**13. What Happens After the RUC Solve**

Once Prescient finishes solving the RUC, it invokes the `after_ruc_generation` callbacks.

In this coordinator, those are:

- `fetch_DA_prices(...)`
- `fetch_DA_dispatches(...)`
- `push_day_ahead_stats_to_forecaster(...)`

registered here:
[`coordinator.py`](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/coordinator.py#L87)

This is how the cleared DA prices/dispatches get brought back into the IDAES side for later RT use.

---

**Compact End-to-End Summary**

1. Prescient loads plugin module
2. plugin module exposes `coordinator.register_plugins`
3. coordinator registers `bid_into_DAM` on `before_ruc_solve`
4. Prescient creates deterministic RUC instance
5. Prescient invokes `bid_into_DAM(options, simulator, ruc_instance, ruc_date, ruc_hour)`
6. non-first day: coordinator projects remaining-day trajectory
7. bidder’s DA model is updated with implemented/projected unit state
8. bidder gets forecast prices from the forecaster
9. bidder solves the DA bidding optimization
10. bidder assembles bid outputs
11. coordinator writes those bid outputs into `ruc_instance.data["elements"]["generator"][gen_name]`
12. Prescient solves the modified RUC and day-ahead market
13. post-RUC callbacks fetch the cleared results back into IDAES

---

**Key Files Involved**

- Plugin entry module:
  [`thermal_generator_prescient_plugin.py`](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/examples/thermal_generator_prescient_plugin.py)

- Main DAM callback registration and logic:
  [`coordinator.py`](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/coordinator.py)

- Bidder model update / solve:
  [`bidder.py`](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/bidder.py)

- Physical generator model state update:
  [`thermal_generator.py`](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/examples/thermal_generator.py)

- Forecaster implementation:
  [`forecaster.py`](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/forecaster.py)

- Prescient callback storage:
  [`internal.py`](/Users/vardhans/Library/Python/3.11/lib/python/site-packages/prescient/plugins/internal.py)

- Prescient plugin registration API:
  [`plugin_registration.py`](/Users/vardhans/Library/Python/3.11/lib/python/site-packages/prescient/plugins/plugin_registration.py)

- Prescient runtime hook invocation:
  [`oracle_manager.py`](/Users/vardhans/Library/Python/3.11/lib/python/site-packages/prescient/simulator/oracle_manager.py)



