---
{"dg-publish":true,"permalink":"/30-projects/double-simulation-loop/tracker/"}
---


`Tracker` is the real-time implementation controller. It takes market dispatch targets and computes a feasible near-term trajectory that your plant can actually follow.

It is in [`/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/tracker.py`](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/tracker.py).

**What information tracker needs**

- `tracking_model_object` (your plant model API: `populate_model`, `update_model`, `get_implemented_profile`, `get_last_delivered_power`, `power_output`, `total_cost`) checked in [`tracker.py:92`](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/tracker.py:92).
- `tracking_horizon`, `n_tracking_hour`, solver.
- Real-time market dispatch signal from coordinator each step.

**How it gets this information**

- Plugin builds `Tracker(...)` and passes model + solver.
- Coordinator builds market signal from SCED/DA and calls:
  `tracker.track_market_dispatch(market_dispatch=..., date=..., hour=...)`
  at [`coordinator.py:749`](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/coordinator.py:749).

**What tracker does with the information**

1. Builds plant model for tracking horizon via `populate_model`.
2. Binds plant market-facing variable (`power_output`) as tracking variable ([`tracker.py:57`](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/tracker.py:57)).
3. Adds dispatch parameter + deviation variables.
4. Enforces tracking balance:
   
$$   P_t + \text{under}_t = D_t + \text{over}_t$$
   
   at [`tracker.py:232`](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/tracker.py:232).
5. Solves optimization and records implemented part of the trajectory.

**What optimization it solves**

Tracker solves a **cost-minimization** problem:
- Objective sense minimize ([`tracker.py:257`](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/tracker.py:257)).
- Includes plant operating cost (`total_cost`) plus penalties on tracking deviations:
  
$$  \min \sum_t \left(w \cdot C_t + \pi_t(\text{under}_t+\text{over}_t)\right)
$$  
  at [`tracker.py:259`](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/tracker.py:259)-[`267`](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/tracker.py:267).

So bidder is profit-max; tracker is feasible implementation min-cost with dispatch-following penalties.

**What it outputs**

Primary output is `implemented_profiles` from your model object:
- returned by `get_implemented_profile(...)` (typically first `n_tracking_hour` steps).
- Then tracker calls `update_model(**profiles)` to roll state forward ([`tracker.py:270`](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/tracker.py:270), [`tracker.py:281`](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/tracker.py:281)).
- It also provides `get_last_delivered_power()` for settlement/report update ([`tracker.py:361`](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/tracker.py:361)).

**Who receives tracker output**

- Coordinator receives `implemented_profiles` and pushes them to:
  - tracker internal state update,
  - bidder RT model update (so bids stay state-consistent),
  in [`coordinator.py:753`](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/coordinator.py:753)-[`756`](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/coordinator.py:756).
- Coordinator sends last delivered power to Prescient ops stats in [`coordinator.py:759`](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/coordinator.py:759)-[`787`](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/coordinator.py:787).


# Tracker explanation in detail from GPT

This is **the most important conceptual question** in the whole IDAES double-loop design. If you understand this clearly, the entire architecture will make sense.

Your intuition is correct:

> _“If the bidder already respects plant physics, why do we need the tracker again?”_

The answer is:

> **Because what you bid and what the market finally dispatches are not the same thing.**

The tracker exists to handle the **difference between your planned schedule and the actual market dispatch.**

Let’s go step by step.

---

## 1️⃣ What the Bidder actually does

When the **Bidder** runs, it solves a planning problem like:

[  
\max \sum_t \lambda_t P_t - C(P_t)  
]

subject to plant physics:

- ramp limits
    
- capacity limits
    
- startup/shutdown
    
- backlog dynamics (for IDC)
    

So yes — the bidder **respects physics**.

But the bidder is only deciding:

> **“If I were free to choose my schedule given price forecasts, what would I want to do?”**

This produces a **bid curve**, not a final dispatch.

Example:

|price|quantity|
|---|---|
|30 $/MWh|40 MW|
|40 $/MWh|70 MW|

---

## 2️⃣ What the market does after that

Prescient then clears the market using **all generators and loads**.

The market might decide:

> “Generator A → 30 MW  
> Generator B → 50 MW  
> Your unit → **65 MW**”

Notice:

**65 MW may not match what you planned.**

Maybe your bidder planned:

```
planned schedule:
hour 10 → 70 MW
```

But market dispatch:

```
actual dispatch:
hour 10 → 65 MW
```

Or worse:

```
planned: 40 → 70 → 90 MW
market:  40 → 80 → 30 MW
```

The market is free to do that.

---

## 3️⃣ Why that causes a physical problem

Your plant physics apply to **actual operation**, not just planning.

Example: ramp constraint

[  
|P_t - P_{t-1}| \le 10  
]

Suppose your plant was at **50 MW**.

Market dispatch:

```
next step → 80 MW
```

But ramp limit allows only:

```
50 → 60
```

So the dispatch is **physically impossible**.

This happens frequently in real markets.

---

## 4️⃣ What the Tracker fixes

Tracker solves a small optimization:

> “Given the market dispatch target, what is the **closest feasible trajectory** my plant can follow?”

Example:

Market says:

```
target = 80 MW
```

Plant constraint:

```
ramp ≤ 10 MW
```

Tracker decides:

```
actual = 60 MW
```

and records:

```
under-delivery = 20 MW
```

---

## 5️⃣ Why this is realistic

Real power systems also have this layer.

There are always **three levels**:

```
Market plan
     ↓
Plant control / feasibility
     ↓
Actual delivered power
```

IDAES models this with:

```
Bidder  → market plan
Tracker → plant control
```

---

## 6️⃣ Another reason tracker exists: state evolution

Bidder solves **a forecast problem**.

But reality changes.

Example for IDC:

Bidder assumed:

```
backlog = 100 jobs
```

But actual operations served:

```
30 jobs
```

Now backlog is:

```
70 jobs
```

Tracker updates that state so the next bid is correct.

Without tracker, the model would drift away from reality.

---

## 7️⃣ Short comparison

|Component|Purpose|Time scale|
|---|---|---|
|**Bidder**|Decide what bids to submit based on price forecasts|planning|
|**Market (Prescient)**|Clear bids and determine dispatch|system-level|
|**Tracker**|Implement dispatch while respecting plant physics|real-time|

---

## 8️⃣ Why this matters for your IDC

For IDC the mismatch will be **even more important**.

Example:

Market dispatch:

```
consume 120 MW now
```

But your IDC has:

- server limits
    
- backlog constraints
    
- cooling limits
    

Tracker will solve:

```
what is the closest feasible compute schedule
```

while respecting:

```
backlog dynamics
server capacity
power limits
```

---

## 9️⃣ One simple sentence to remember

**Bidder = what you would like to do.  
Tracker = what you can actually do.**

---

If you want, I can also show you something extremely useful next:

**a concrete numeric example (with numbers) showing exactly how bidder plan, market dispatch, and tracker implementation differ.**

That example usually makes the whole architecture click instantly.