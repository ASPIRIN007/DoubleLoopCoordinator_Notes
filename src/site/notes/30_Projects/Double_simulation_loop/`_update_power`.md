---
{"dg-publish":true,"permalink":"/30-projects/double-simulation-loop/update-power/"}
---


## Code

def _update_power(b, implemented_power_output):

<span style="color:rgb(255, 192, 0)">This method updates the parameters in the ramping constraints based on<br>the implemented power outputs.<br>  <br>Arguments:<br>b: the block that needs to be updated<br>implemented<i>power</i>output: realized power outputs: []<br><br>Returns:None</span>

b.pre_P_T = round(implemented_power_output[-1], 2)
b.pre_on_off = round(int(implemented_power_output[-1] > 1e-3))

return


## Explanation
This handles the generator's **momentum**.

- **The Goal:** To satisfy **Ramping Constraints**.
- **The Method:** It looks at the very last power value (`implemented_power_output[-1]`).
- **Why split it?** This is purely about the "position" of the turbine. If the turbine is at 50MW, the next optimization needs that 50MW as a starting point so it doesn't calculate a jump to 100MW that the machine can't physically handle.