---
{"dg-publish":true,"permalink":"/30-projects/double-simulation-loop/compute-day-ahead-bids/","created":"2026-03-15T17:51:37.296-04:00","updated":"2026-03-16T13:41:36.385-04:00"}
---

## Code

def compute_day_ahead_bids(self, date, hour=0):

<span style="color:rgb(255, 192, 0)">Solve the model to bid into the day-ahead market. After solving, record the bids from the solve.<br><br>Arguments:<br>date: current simulation date<br>hour: current simulation hour<br><br>Returns:<br>dict: the obtained bids</span>

(
day_ahead_price, 
real_time_energy_price,
) = self.forecaster.forecast_day_ahead_and_real_time_prices
(
date=date,
hour=hour,
bus=self.bidding_model_object.model_data.bus,
horizon=self.day_ahead_horizon,
n_samples=self.n_scenario,
)
return self._compute_bids
(
day_ahead_price=day_ahead_price,
real_time_energy_price=real_time_energy_price,
date=date,
hour=hour,
model=self.day_ahead_model,
power_var_name="day_ahead_power",
energy_price_param_name="day_ahead_energy_price",
market="Day-ahead",
)

## Explanation 

self.forecaster.forecaster_day_ahead_and_real_time_prices refers us to forecaster.py and to [[30_Projects/Double_simulation_loop/PlaceHolderForecaster\|PlaceHolderForecaster]] and there we calculate the market prices predicted using normal distribution and then returned to [[30_Projects/Double_simulation_loop/compute_bids\|compute_bids]]. 
