---
{"dg-publish":true,"permalink":"/30-projects/double-simulation-loop/prescient-plugins-internal-py/","created":"2026-03-12T23:17:54.572-04:00","updated":"2026-03-16T13:43:33.661-04:00"}
---

The comment on top " Code that is not intended to be called by plugins " is basically saying:

“Plugins should talk to Prescient through PluginRegistrationContext. This file is the behind-the-scenes implementation, not the official front door.”

[[30_Projects/Double_simulation_loop/StatisticsSubscribers class\|StatisticsSubscribers class]]
[[30_Projects/Double_simulation_loop/PluginCallbackManager\|PluginCallbackManager]]

