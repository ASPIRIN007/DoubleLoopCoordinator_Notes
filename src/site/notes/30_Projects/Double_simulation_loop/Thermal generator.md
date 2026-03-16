---
{"dg-publish":true,"permalink":"/30-projects/double-simulation-loop/thermal-generator/","created":"2026-03-01T16:44:56.212-05:00","updated":"2026-03-16T13:45:24.191-04:00"}
---

The thermal-generator Eq. (1)–(12) from that paper are implemented in the `ThermalGenerator` Pyomo model here:
#gao #thermal_gen_gao
- [thermal_generator.py](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/examples/thermal_generator.py)

Use this mapping

1. **Eq. (1)** (binary commitment/state variables):  
   [thermal_generator.py:334](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/examples/thermal_generator.py#L334), [thermal_generator.py:337](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/examples/thermal_generator.py#L337), [thermal_generator.py:340](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/examples/thermal_generator.py#L340)

2. **Eq. (2)** (startup/shutdown/commitment transition logic):  
   [thermal_generator.py:386](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/examples/thermal_generator.py#L386) to [thermal_generator.py:392](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/examples/thermal_generator.py#L392)

3. **Eq. (3)** (cannot startup and shutdown simultaneously):  
   [thermal_generator.py:395](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/examples/thermal_generator.py#L395) to [thermal_generator.py:398](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/examples/thermal_generator.py#L398)

4. **Eq. (4)** (ramp-up constraint):  
   [thermal_generator.py:401](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/examples/thermal_generator.py#L401) to [thermal_generator.py:420](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/examples/thermal_generator.py#L420)

5. **Eq. (5)** (shutdown ramp constraint):  
   [thermal_generator.py:423](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/examples/thermal_generator.py#L423) to [thermal_generator.py:434](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/examples/thermal_generator.py#L434)

6. **Eq. (6)** (ramp-down constraint):  
   [thermal_generator.py:437](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/examples/thermal_generator.py#L437) to [thermal_generator.py:452](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/examples/thermal_generator.py#L452)

7. **Eq. (7)** (minimum down-time):  
   [thermal_generator.py:209](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/examples/thermal_generator.py#L209) to [thermal_generator.py:228](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/examples/thermal_generator.py#L228)

8. **Eq. (8)** (minimum up-time):  
   [thermal_generator.py:230](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/examples/thermal_generator.py#L230) to [thermal_generator.py:249](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/examples/thermal_generator.py#L249)

9. **Eq. (9)** (total power = piecewise segments + min-load part):  
   [thermal_generator.py:361](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/examples/thermal_generator.py#L361) to [thermal_generator.py:367](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/examples/thermal_generator.py#L367)

10. **Eq. (10)** (piecewise segment production variables/constraints):  
   [thermal_generator.py:343](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/examples/thermal_generator.py#L343), [thermal_generator.py:370](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/examples/thermal_generator.py#L370) to [thermal_generator.py:383](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/examples/thermal_generator.py#L383)

11. **Eq. (11)** (capacity/interval bounds used with piecewise formulation):  
   [thermal_generator.py:350](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/examples/thermal_generator.py#L350) to [thermal_generator.py:358](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/examples/thermal_generator.py#L358), plus [thermal_generator.py:370](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/examples/thermal_generator.py#L370) to [thermal_generator.py:383](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/examples/thermal_generator.py#L383)

12. **Eq. (12)** (startup cost term):  
   [thermal_generator.py:466](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/examples/thermal_generator.py#L466) to [thermal_generator.py:469](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/examples/thermal_generator.py#L469)

Also note the file explicitly cites your paper’s thermal model in its class docstring: [thermal_generator.py:28](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/examples/thermal_generator.py#L28).