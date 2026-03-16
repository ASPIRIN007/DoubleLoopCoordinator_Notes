---
{"dg-publish":true,"permalink":"/30-projects/double-simulation-loop/update-ut-dt/","created":"2026-03-15T18:19:51.483-04:00","updated":"2026-03-16T13:40:47.773-04:00"}
---

This method updates the parameters in the minimum up/down time constraints based on the implemented shut down and start up events.

## Code

``def _update_UT_DT(b, implemented_shut_down, implemented_start_up):

<span style="color:rgb(255, 192, 0)">This method updates the parameters in the minimum up/down time<br>constraints based on the implemented shut down and start up events.<br><br>Arguments:<br>b: the block that needs to be updated<br>implemented<i>shut</i>down: realized shut down events: [ ]<br>implemented<i>start</i>up: realized start up events: [ ]<br><br>Returns: None</span>

pre_shut_down_trajectory_copy = [[20_Concepts/Python notebook/deque\|deque]]([ ])

pre_start_up_trajectory_copy = [[20_Concepts/Python notebook/deque\|deque]]([ ])

<span style="color:rgb(255, 192, 0)">copy old trajectory</span>

for t in b.pre_shut_down_trajectory_set:
pre_shut_down_trajectory_copy.append(round(pyo.value(b.pre_shut_down_trajectory[t])))

for t in b.pre_start_up_trajectory_set:
pre_start_up_trajectory_copy.append(round(pyo.value(b.pre_start_up_trajectory[t])))

<span style="color:rgb(255, 192, 0)">add implemented trajectory to the queue</span>

pre_shut_down_trajectory_copy += deque(implemented_shut_down)
pre_start_up_trajectory_copy += deque(implemented_start_up)  
<span style="color:rgb(255, 192, 0)"><br>pop out outdated trajectory</span>

while len(pre_shut_down_trajectory_copy) > pyo.value(b.min_dw_time) - 1:
pre_shut_down_trajectory_copy.popleft()

while len(pre_start_up_trajectory_copy) > pyo.value(b.min_up_time) - 1:
pre_start_up_trajectory_copy.popleft()  

<span style="color:rgb(255, 192, 0)">actual update</span>

for t in b.pre_shut_down_trajectory_set:
b.pre_shut_down_trajectory[t] = pre_shut_down_trajectory_copy.popleft()

for t in b.pre_start_up_trajectory_set:
b.pre_start_up_trajectory[t] = pre_start_up_trajectory_copy.popleft()

return``


## What does deque do in `update_UT_DT`?

Power plant models need to track how long a generator has been **ON** (Up Time) or **OFF** (Down Time) because physical boilers can't be toggled instantly; they have "Minimum Up Time" and "Minimum Down Time" constraints.

The `deque` is used here to create a **sliding window of history**.

### 2. The "Sliding Window" Behavior

When you define a `deque` with a `maxlen`, it automatically manages its own size:

- If you add a new value to a "full" deque, the **oldest value is automatically kicked out** at the other end.
- In the thermal generator plugin, this is used to keep track of the last $X$ hours of generator status.

**Example:**

Imagine a generator has a Minimum Up Time of 3 hours. We use a `deque(maxlen=3)`.

1. **Hour 1:** Generator turns ON. Deque: `[ON]`
2. **Hour 2:** Still ON. Deque: `[ON, ON]`
3. **Hour 3:** Still ON. Deque: `[ON, ON, ON]`
4. **Hour 4:** Still ON. The first `ON` is kicked out, replaced by the new one. Deque: `[ON, ON, ON]`

### 3. Why use `deque` instead of a regular `list`?

In the `update_UT_DT` function, efficiency and "self-cleaning" are key:

- **Automatic Cleanup:** With a list, you would have to manually write code to delete the first element every time you add a new one to keep the size constant. `deque` does this automatically.
- **Performance:** Adding or removing items from the ends of a `deque` is much faster ($O(1)$ complexity) than a list ($O(n)$ complexity).
- **Memory:** It ensures the "history" never grows too large and consumes too much RAM, which is important for long simulations (like a full year).