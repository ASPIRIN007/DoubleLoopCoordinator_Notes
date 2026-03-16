---
{"dg-publish":true,"permalink":"/30-projects/double-simulation-loop/pass-da-bid-to-prescient/","created":"2026-03-15T17:23:59.915-04:00","updated":"2026-03-16T13:40:03.093-04:00"}
---

## code

`def _pass_DA_bid_to_prescient(self, options, ruc_instance, bids):`

<span style="color:rgb(255, 192, 0)">This method passes the bids into the RUC model for day-ahead market clearing.<br>Arguments:<br>options: Prescient options from prescient.simulator.config.<br>ruc_instance: Prescient RUC object<br>bids: the bids we want to pass into the day-ahead market.<br>Returns:None</span>

gen_name = self.bidder.bidding_model_object.model_data.gen_name

<span style="color:rgb(255, 192, 0)">fetch the generator's parameter dictionary from Prescient UC instance</span>

`gen_dict = ruc_instance.data["elements"]["generator"][gen_name]`
`self._update_bids(gen_dict, bids, start_hour=0, horizon=options.ruc_horizon)`

return

## explanation

`_pass_DA_bid_to_prescient(...)` is taking the bid dictionary produced by the IDAES bidder and writing it into Prescient’s in-memory RUC generator data for the one controlled generator.

It is here:
[`coordinator.py`](file:///Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/coordinator.py#L272)

What it does step by step:

1. It gets the controlled generator name:
- `gen_name = self.bidder.bidding_model_object.model_data.gen_name`

2. It looks up that generator inside the Prescient/Egret RUC model data:
- `ruc_instance.data["elements"]["generator"][gen_name]`
- Source: [`coordinator.py`](file:///Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/coordinator.py#L289)

3. It calls `_update_bids(...)` on that generator dictionary:
- Source: [`coordinator.py`](file:///Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/coordinator.py#L292)

`_update_bids(...)` then overwrites fields in that generator’s data such as:

- `p_cost`
- `p_max`
- `p_min`
- `fixed_commitment`
- `startup_capacity`
- `shutdown_capacity`
- `startup_cost`
- `min_up_time`
- `min_down_time`

So this is how the DA bid is “passed to Prescient”:
it is not sent through a special Prescient API call or message queue.
It is written directly into the generator data dictionary inside the `ruc_instance` that Prescient is about to solve.

Where it goes in Prescient:

This happens during the `before_ruc_solve` callback stage. Prescient creates the deterministic RUC instance, then invokes your callback:
[`oracle_manager.py`](file:///Users/vardhans/Library/Python/3.11/lib/python/site-packages/prescient/simulator/oracle_manager.py#L144)

So the sequence is:

1. Prescient creates `deterministic_ruc_instance`
2. Prescient invokes `before_ruc_solve` callbacks
3. your `bid_into_DAM(...)` runs
4. `bid_into_DAM(...)` calls `_pass_DA_bid_to_prescient(...)`
5. that mutates `deterministic_ruc_instance.data["elements"]["generator"][gen_name]`
6. Prescient then solves that modified RUC instance

So the bid “goes into Prescient” by changing the RUC model data before the solver is called.

In short:
- `_pass_DA_bid_to_prescient` writes your bid into the Prescient RUC generator record
- Prescient then uses that modified record when it solves the day-ahead RUC and market

That is why this hook is registered on `register_before_ruc_solve_callback(...)`: it needs to modify the RUC inputs just before Prescient solves them.