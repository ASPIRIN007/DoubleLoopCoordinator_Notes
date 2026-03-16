---
{"dg-publish":true,"permalink":"/30-projects/double-simulation-loop/project-tracking-trajectory/","created":"2026-03-15T17:23:01.530-04:00","updated":"2026-03-16T13:44:21.387-04:00"}
---

What project_tracking_trajectory(...) does:

1. It reads the current simulation date/hour from Prescient.
2. It clones the current tracker state into projection_tracker.
3. It loops from ruc_execution_hour to 23
4. For each of those hours, it builds a projected tracking signal and solves the projection tracker.
5. It merges:
    - actual tracker daily stats so far
    - projected tracker daily stats for the rest of the day
6. It returns that merged trajectory as full_projected_trajectory.