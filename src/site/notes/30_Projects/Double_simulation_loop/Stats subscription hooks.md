---
{"dg-publish":true,"permalink":"/30-projects/double-simulation-loop/stats-subscription-hooks/"}
---

Examples:
- `register_for_operations_stats`
- `register_for_hourly_stats`
- `register_for_daily_stats`
- `register_for_overall_stats`

These are handled differently.

They first go into pending subscriber lists in [[30_Projects/Double_simulation_loop/StatisticsSubscribers class\|StatisticsSubscribers class]] and then during initialization [[30_Projects/Double_simulation_loop/PluginCallbackManager\|PluginCallbackManager]] calls it, specifically:
- `_pending_stats_subscribers.operations`
- `_pending_stats_subscribers.hourly`
- `_pending_stats_subscribers.daily`
- `_pending_stats_subscribers.overall`

Then during initialization, Prescient moves them into [[30_Projects/Double_simulation_loop/prescient-simulator-stats_manager.py\|prescient-simulator-stats_manager.py]]

So for hourly stats, your example is exactly right:
- first registered into the pending hourly subscriber list
- later attached to `stats_manager`
- later called when hourly stats are published