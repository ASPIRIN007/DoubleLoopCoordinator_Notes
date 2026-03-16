---
{"dg-publish":true,"permalink":"/30-projects/double-simulation-loop/place-holder-forecaster/","created":"2026-03-15T19:11:24.552-04:00","updated":"2026-03-16T13:42:56.231-04:00"}
---

PlaceHolderForecaster is what is being used in thermal_generator_prescient_plugin.py. This is named PlaceHolder because currently it is just using the means and STDs to forecast the future market prices based on which bidder then create bids. Since just using mean and STDs are not a good prediction it can be swapped later with a better forecaster and thus named placeholder.

## code

class PlaceHolderForecaster(AbstractPrescientPriceForecaster):

<span style="color:rgb(255, 192, 0)">T</span><span style="color:rgb(255, 192, 0)">his a placeholder for a real price forecaster. This placeholder can takes<br>representative daily values and standard deviations for real-time and<br>day-ahead prices to forecast prices in the future.<br></span>
def __init__(self,
daily_da_price_means: list, 
daily_rt_price_means: list, 
daily_da_price_stds: list,
daily_rt_price_stds: list, ):

<span style="color:rgb(255, 192, 0)">Initialize the PlaceHolderForecaster.<br>Arguments:<br>daily<i>da</i>price_means: list of day-ahead price means<br>daily<i>rt</i>price_means: list of real-time price means<br>daily<i>da</i>price_stds: list of price standard deviations<br>daily<i>rt</i>price_stds: list of real-time price standard deviations<br>Returns:Non</span><span style="color:rgb(255, 192, 0)">e</span>

self.daily_da_price_means = daily_da_price_means
self.daily_rt_price_means = daily_rt_price_means
self.daily_da_price_stds = daily_da_price_stds
self.daily_rt_price_stds = daily_rt_price_stds

---

def forecast_day_ahead_and_real_time_prices(self, date, hour, bus, horizon, n_samples):

<span style="color:rgb(255, 192, 0)">Forecast both day-ahead and real-time market prices.<br>Arguments:<br>date: intended date of the forecasts<br>hour: intended hour of the forecasts<br>bus: intended bus of the forecasts<br>horizon: number of the time periods of the forecasts<br>n_samples: number of the samples<br>Returns:<br>dict: day-ahead price forecasts<br>dict: real-time price forecasts</span>

rt_forecast = self.forecast_real_time_prices(date, hour, bus, horizon, n_samples)
da_forecast = self.forecast_day_ahead_prices(date, hour, bus, horizon, n_samples)
return 
da_forecast, rt_forecast

---

def forecast_day_ahead_prices(self, date, hour, bus, horizon, n_samples):


<span style="color:rgb(255, 192, 0)">Forecast day-ahead market prices.<br>Arguments:<br>date: intended date of the forecasts<br>hour: intended hour of the forecasts<br>bus: intended bus of the forecasts<br>horizon: number of the time periods of the forecasts<br>n_samples: number of the samples<br>Returns:<br>dict: day-ahead price forecasts</span>

return self._forecast(
means=self.daily_da_price_means,
stds=self.daily_da_price_stds,
hour=hour,
horizon=horizon,
n_samples=n_samples,)

---

def forecast_real_time_prices(self, date, hour, bus, horizon, n_samples):

<span style="color:rgb(255, 192, 0)">Forecast real-time market prices.<br>Arguments:<br>date: intended date of the forecasts<br>hour: intended hour of the forecasts<br>bus: intended bus of the forecasts<br>horizon: number of the time periods of the forecasts<br>n_samples: number of the samples<br>Returns:<br>dict: real-time price forecasts</span>

return self._forecast(
means=self.daily_rt_price_means,
stds=self.daily_rt_price_stds,
hour=hour,
horizon=horizon,
n_samples=n_samples,)

---

def _forecast(self, means, stds, hour, horizon, n_samples):

<span style="color:rgb(255, 192, 0)">Generate price forecasts.<br>Arguments:<br>means: list of price means<br>stds: list of price standard deviations<br>hour: intended hour of the forecasts<br>horizon: number of the time periods of the forecasts<br>n_samples: number of the samples<br>Returns:<br>dict: real-time price forecasts</span>

corresponding_means = [means[t % 24] for t in range(hour, hour + horizon)]
corresponding_stds = [stds[t % 24] for t in range(hour, hour + horizon)]
forecasts_arr = np.random.normal(loc=corresponding_means, scale=corresponding_stds, size=(n_samples, horizon))
forecasts_arr[forecasts_arr < 0] = 0
return {i: list(forecasts_arr[i]) for i in range(n_samples)}

---
def fetch_hourly_stats_from_prescient(self, prescient_hourly_stats):

<span style="color:rgb(255, 192, 0)">This method fetches the hourly stats from Prescient to the price forecaster<br>once the hourly stats are published.<br>Arguments:<br>prescient<i>hourly</i>stats: Prescient HourlyStats object.<br>Returns:<br>None</span>

return

---

def fetch_day_ahead_stats_from_prescient(self, uc_date, uc_hour, day_ahead_result):

<span style="color:rgb(255, 192, 0)">This method fetches the day-ahead market to the price forecaster after the<br>UC is solved from Prescient through the coordinator.<br>Arguments:<br>ruc_date: the date of the day-ahead market we bid into.<br>ruc_hour: the hour the RUC is being solved in the day before.<br>day<i>ahead</i>result: a Prescient RucPlan object.<br>Returns:<br>None</span>

return

---

## Explanation

`PlaceHolderForecaster` is a simple dummy price forecaster. It is not learning from the market in a sophisticated way; it just generates forecast price scenarios from pre-specified daily means and standard deviations.

**What it is for**

It gives the bidder some price forecasts to work with, without requiring a real forecasting model.
it is basically a stand-in forecaster simple enough to run without building a full forecasting pipeline.

**What inputs it stores**

When created, it takes four arrays/lists:
- `daily_da_price_means`
- `daily_rt_price_means`
- `daily_da_price_stds`
- `daily_rt_price_stds`

In the thermal generator example, those come from [[30_Projects/Double_simulation_loop/Thermal_generator_prescient_plugin.py\|Thermal_generator_prescient_plugin.py]]
So the forecaster has:
- a representative mean DA price for each hour of day
- a representative std dev for each hour of day
- similarly for RT prices

**Main methods**

`forecast_day_ahead_and_real_time_prices(...)`
- calls:
  - `forecast_day_ahead_prices(...)`
  - `forecast_real_time_prices(...)`
- returns both DA and RT forecasts

`forecast_day_ahead_prices(...)`
- uses the stored DA means/stds

`forecast_real_time_prices(...)`
- uses the stored RT means/stds

Both eventually call `_forecast(...)`.

**What `_forecast(...)` does conceptually**

It generates `n_samples` price scenarios over the requested `horizon`, using the hourly means and std deviations for the corresponding hours.

So for each scenario:
- for each time step in the horizon
- it picks the appropriate hour-of-day mean/std
- and samples a price

So the output is not one price, but a set of scenario price trajectories.

**What it does not do**

It does not really use:
- current Prescient state in a complex way
- recent hourly stats to update itself
- advanced ML/statistical fitting
- adaptive market feedback in a sophisticated sense

That is why it is called “<span style="color:rgb(255, 192, 0)">placeholder</span>.”

**How it relates to Prescient callbacks**

Because it subclasses `AbstractPrescientPriceForecaster`, it also has methods like:
- `fetch_hourly_stats_from_prescient(...)`
- `fetch_day_ahead_stats_from_prescient(...)`

But for `PlaceHolderForecaster`, those methods are basically placeholders/no-op style behavior compared with more sophisticated forecasters. Its real forecasting logic still comes from the fixed daily mean/std inputs.

**Why it is useful**

It lets the bidder solve a stochastic bidding problem with price scenarios, even when you do not yet have a proper market forecasting model.
