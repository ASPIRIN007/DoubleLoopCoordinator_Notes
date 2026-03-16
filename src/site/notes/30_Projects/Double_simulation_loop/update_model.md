---
{"dg-publish":true,"permalink":"/30-projects/double-simulation-loop/update-model/"}
---

## code

def update_model(self, b, implemented_shut_down, implemented_start_up, implemented_power_output):

<span style="color:rgb(255, 192, 0)">This method updates the parameters in the model based on the implemented power outputs, shut down and start up events.<br><br>Arguments:<br>b: the block that needs to be updated<br>implemented<i>shut</i>down: realized shut down events: [ ].<br>implemented<i>start</i>up: realized start up events: [ ]<br>implemented<i>power</i>output: realized power outputs: [ ]<br>  <br>Returns:<br>None</span>

self.[[`_update_UT_DT`]](b, implemented_shut_down, implemented_start_up)
self.[[`_update_power`]](b, implemented_power_output)


## Explanation

They have broken down the update_model into two parts `_update_power` and `_update_UT_DT`.
The update_power handles the ramping rate part while update_UT_DT handles the minimum uptime and downtime part.
### Why this structure is professional

By separating them, the developers made the code **Modular**. If they wanted to create a new type of generator (like a Battery) that has ramping limits but _doesn't_ have minimum up/down times, they could simply reuse `_update_power` and ignore `_update_UT_DT`.

