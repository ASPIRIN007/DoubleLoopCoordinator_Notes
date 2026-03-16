---
{"dg-publish":true,"permalink":"/30-projects/double-simulation-loop/invoke-initialization-callbacks/","created":"2026-03-12T23:18:06.187-04:00","updated":"2026-03-16T13:42:25.813-04:00"}
---


def invoke_initialization_callbacks(self, options, simulator):

        for cb in self._initialization_callbacks:
            cb(options, simulator)

        # register stats callbacks as subscribers

        for s in self._pending_stats_subscribers.operations:
            simulator.stats_manager.register_for_sced_stats(s)
        self._pending_stats_subscribers.operations.clear()

        for s in self._pending_stats_subscribers.hourly:
            simulator.stats_manager.register_for_hourly_stats(s)
        self._pending_stats_subscribers.hourly.clear()

        for s in self._pending_stats_subscribers.daily:
            simulator.stats_manager.register_for_daily_stats(s)
        self._pending_stats_subscribers.daily.clear()

        for s in self._pending_stats_subscribers.overall:
            simulator.stats_manager.register_for_overall_stats(s)
        self._pending_stats_subscribers.overall.clear()


What is happning in this invoke_initialization_callbacks?

Here if I explain say this piece of code
        for s in self._pending_stats_subscribers.hourly:
            simulator.stats_manager.register_for_hourly_stats(s)
        self._pending_stats_subscribers.hourly.clear()

It takes all hourly stats subscriber callbacks that were queued up during plugin registration, registers them with the simulator’s stats manager, then clears the pending list so the same callbacks aren’t registered again on subsequent initialization calls (and to drop the temporary references).

It does the same process for different subscribers class 
- hourly subscribers
- daily subscribers
- operations subscribers
- overall subscribers 