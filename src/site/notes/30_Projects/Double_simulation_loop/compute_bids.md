---
{"dg-publish":true,"permalink":"/30-projects/double-simulation-loop/compute-bids/","created":"2026-03-15T20:01:48.235-04:00","updated":"2026-03-16T13:41:32.183-04:00"}
---


## Code

def _compute_bids(
self,
day_ahead_price,
real_time_energy_price,
date,
hour,
model,
power_var_name,
energy_price_param_name,
market,
):

<span style="color:rgb(255, 192, 0)">Solve the model to bid into the markets. After solving, record the bids from the solve.<br>Arguments:<br>day<i>ahead</i>price: day-ahead price forecasts needed to solve the bidding problem<br>real<i>time</i>energy_price: real-time price forecasts needed to solve the bidding problem<br>date: current simulation date<br>hour: current simulation hour<br>model: bidding model<br>power<i>var</i>name: the name of the power output (str)<br>energy<i>price</i>param_name: the name of the energy price forecast params (str)<br>market: the market name (str), e.g., Day-ahead, real-time<br>  <br>Returns:<br>dict: the obtained bids<br><br>update the price forecasts</span>

self.[[30_Projects/Double_simulation_loop/_pass_price_forecasts\|_pass_price_forecasts]](model, day_ahead_price, real_time_energy_price)
self.solver.solve(model, tee=True)
bids = self._assemble_bids(
model,
power_var_name=power_var_name,
energy_price_param_name=energy_price_param_name,
hour=hour,
)
self.record_bids(bids, model=model, date=date, hour=hour, market=market)
return bids

## Explanation

`_compute_bids(...)` is the common “solve-and-extract-bids” routine for both DA and RT bidding.

It does four main things in order:
[`bidder.py`](file:///Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/bidder.py#L456)

1. **Load forecast prices into the bidding model**
- via `_pass_price_forecasts(...)`
- this writes the DA/RT price forecasts into each scenario block’s model parameters

2. **Solve the optimization model**
- `self.solver.solve(model, tee=True)`

3. **Assemble the bid data from the solved model**
- via `_assemble_bids(...)`
- this reads the optimized power/cost information and builds the bid structure that will be passed back to Prescient

4. **Record results**
- via `record_bids(...)`
- this saves both the assembled bids and detailed model results

Then it returns the final `bids` dictionary.

So conceptually `_compute_bids(...)` is the engine of the bidder:

- put in forecasted prices
- solve the model
- read off the economically optimal offer
- package it as bids

Why it exists:
both `compute_day_ahead_bids(...)` and `compute_real_time_bids(...)` need the same basic workflow. The difference is just:
- which model they solve
- which prices they pass in
- which power variable represents the offered quantity
- whether it is DA or RT market

That’s why both methods call `_compute_bids(...)` with different arguments.

So the short version is:

`_compute_bids(...)` takes forecast prices and a bidding model, solves the optimization, converts the solution into bid objects, records them, and returns them.

