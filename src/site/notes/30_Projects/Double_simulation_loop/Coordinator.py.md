---
{"dg-publish":true,"permalink":"/30-projects/double-simulation-loop/coordinator-py/","created":"2026-03-12T20:42:00.680-04:00","updated":"2026-03-16T13:41:48.041-04:00"}
---

class PrescientPluginModule(ModuleType):

	def __init__(self, get_configuration, register_plugins):

		self.get_configuration = get_configuration

		self.register_plugins = register_plugins

# class DoubleLoopCoordinator:
## def __init__(self, bidder, tracker, projection_tracker):
		self.bidder = bidder
		self.tracker = tracker
		self.projection_tracker = projection_tracker
## def register_plugins(self, context, options, plugin_config):
[[30_Projects/Double_simulation_loop/register_plugins\|register_plugins]]

<span style="color:rgb(0, 112, 192)">Register functionalities in Prescient's plugin system.<br>Arguments:<br>context: Prescient plugin PluginRegistrationContext from prescient.plugins.plugin_registration<br>options: Prescient options from prescient.simulator.config<br>plugin_config: Prescient plugin config<br>Returns:None</span>

self.plugin_config = plugin_config

context.[[30_Projects/Double_simulation_loop/register_initialization_callback\|register_initialization_callback]](self.[[30_Projects/Double_simulation_loop/initialize_customized_results\|initialize_customized_results]])

context.[[30_Projects/Double_simulation_loop/register_for_hourly_stats\|register_for_hourly_stats]](self.[[30_Projects/Double_simulation_loop/push_hourly_stats_to_forecaster\|push_hourly_stats_to_forecaster]])

context.[[30_Projects/Double_simulation_loop/register_after_get_initial_actuals_model_for_sced_callback\|register_after_get_initial_actuals_model_for_sced_callback]](self.update_static_params)

context.register_after_get_initial_actuals_model_for_simulation_actuals_callback(self.update_static_params)

context.register_after_get_initial_forecast_model_for_ruc_callback(self.update_static_params)

context.register_before_ruc_solve_callback(self.[[30_Projects/Double_simulation_loop/bid_into_DAM\|bid_into_DAM]])

context.register_after_ruc_generation_callback(self.fetch_DA_prices)

context.register_after_ruc_generation_callback(self.fetch_DA_dispatches)

context.register_after_ruc_generation_callback(self.push_day_ahead_stats_to_forecaster)

context.register_before_operations_solve_callback(self.bid_into_RTM)

context.register_after_operations_callback(self.track_sced_signal

context.register_after_ruc_activation_callback(self.activate_pending_DA_data)

context.register_finalization_callback(self.write_plugin_results)

return


1. [[__init__\|__init__]]
2. [[30_Projects/Double_simulation_loop/register_plugins\|register_plugins]]
3. [[get_configuration\|get_configuration]]
4. [[30_Projects/Double_simulation_loop/initialize_customized_results\|initialize_customized_results]]
5. [[30_Projects/Double_simulation_loop/push_hourly_stats_to_forecaster\|push_hourly_stats_to_forecaster]]
6. [[push_day_ahead_stats_to_forecaster\|push_day_ahead_stats_to_forecaster]]
7. [[_update_bids\|_update_bids]]
8. [[30_Projects/Double_simulation_loop/_pass_DA_bid_to_prescient\|_pass_DA_bid_to_prescient]]
9. [[assemble_project_tracking_signal\|assemble_project_tracking_signal]]
10. [[30_Projects/Double_simulation_loop/project_tracking_trajectory\|project_tracking_trajectory]]
11. [[_clone_tracking_model\|_clone_tracking_model]]
12. [[_update_static_params\|_update_static_params]]
13. [[update_static_params\|update_static_params]]
14. [[30_Projects/Double_simulation_loop/bid_into_DAM\|bid_into_DAM]]
15. [[fetch_DA_prices\|fetch_DA_prices]]
16. [[fetch_DA_dispatches\|fetch_DA_dispatches]]
17. [[_pass_RT_bid_to_prescient\|_pass_RT_bid_to_prescient]]
18. [[30_Projects/Double_simulation_loop/bid_into_RTM\|bid_into_RTM]]
19. [[assemble_sced_tracking_market_signals\|assemble_sced_tracking_market_signals]]
20. [[_assemble_sced_tracking_market_signals\|_assemble_sced_tracking_market_signals]]
21. [[track_sced_signal\|track_sced_signal]]
22. [[update_observed_dispatch\|update_observed_dispatch]]
23. [[30_Projects/Double_simulation_loop/activate_pending_DA_data\|activate_pending_DA_data]]
24. [[write_plugin_results\|write_plugin_results]]