---
{"dg-publish":true,"permalink":"/30-projects/double-simulation-loop/register-for-hourly-stats/"}
---

The flow is:

1. our plugin calls  
    context.register_for_hourly_stats(my_callback)
    
2. Prescient stores that function in a pending list. 
    
3. Later, during simulator startup, [[30_Projects/Double_simulation_loop/prescient-simulator-simulator.py\|prescient-simulator-simulator.py]] calls: [[30_Projects/Double_simulation_loop/invoke_initialization_callbacks\|invoke_initialization_callbacks]](options, self)
    
4. Inside [[30_Projects/Double_simulation_loop/prescient-plugins-internal.py\|prescient-plugins-internal.py]], that method does two things:
    - it runs the regular initialization callbacks
    - then it moves the pending hourly subscribers into [[30_Projects/Double_simulation_loop/prescient-simulator-stats_manager.py\|prescient-simulator-stats_manager.py]]
    
5. After that, those subscribers are now registered inside [[30_Projects/Double_simulation_loop/prescient-simulator-stats_manager.py\|prescient-simulator-stats_manager.py]]
    
6. Later, when Prescient finishes an hour and publishes hourly stats, <span style="color:rgb(0, 112, 192)">stats_manager</span> actually calls those subscriber functions and publish [[30_Projects/Double_simulation_loop/hourly_stats\|hourly_stats]] to them.
