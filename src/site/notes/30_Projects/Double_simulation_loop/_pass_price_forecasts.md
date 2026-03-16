---
{"dg-publish":true,"permalink":"/30-projects/double-simulation-loop/pass-price-forecasts/"}
---

## Code

def _pass_price_forecasts(self, model, day_ahead_price, real_time_energy_price):

<span style="color:rgb(255, 192, 0)">Pass the price forecasts into model parameters.<br>Arguments:<br>day<i>ahead</i>price: day-ahead price forecasts needed to solve the bidding problem<br>real<i>time</i>energy_price: real-time price forecasts needed to solve the bidding problem<br>Returns:None</span>

for i in model.SCENARIOS:
	time_index = model.fs[i].real_time_energy_price.index_set()
	if day_ahead_price is not None:
		for t, p in zip(time_index, day_ahead_price[i]):
			model.fs[i].day_ahead_energy_price[t] = p
	for t, p in zip(time_index, real_time_energy_price[i]):
		model.fs[i].real_time_energy_price[t] = p

return

## Explanation

`_pass_price_forecasts(...)` writes the forecasted electricity prices into the bidder’s Pyomo model parameters before the optimization is solved.

It is here:
[`bidder.py`](file:///Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/bidder.py#L724)

What it does:

- loops over each scenario block in the stochastic bidding model
- for each scenario, gets the model time index
- writes:
  - `day_ahead_price` into `model.fs[i].day_ahead_energy_price[t]` if DA prices were provided
  - `real_time_energy_price` into `model.fs[i].real_time_energy_price[t]`

So it is basically taking the forecast output from the forecaster and injecting it into the optimization model so the objective/constraints use those prices.

In plain words:

- the forecaster predicts price scenarios
- `_pass_price_forecasts(...)` loads those scenarios into the bidder model
- then `_compute_bids(...)` solves the model using those prices

So it does not compute forecasts.
It only transfers already-computed forecasts into the model parameters.

A compact interpretation:

- forecaster = “here are the predicted prices”
- `_pass_price_forecasts` = “write those predicted prices into the model”
- solver = “now optimize bids using those prices”