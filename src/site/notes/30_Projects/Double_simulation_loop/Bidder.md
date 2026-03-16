---
{"dg-publish":true,"permalink":"/30-projects/double-simulation-loop/bidder/"}
---



`Bidder` is a stochastic optimization agent that computes offer curves for DA and RT.

It is implemented in [`/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/bidder.py`](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/bidder.py).

**What information bidder needs**

It needs:
- A model object (`bidding_model_object`) with plant physics/cost via `populate_model`, `power_output`, `total_cost`, `model_data` ([`bidder.py:173`](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/bidder.py:173)).
- Horizons (`day_ahead_horizon`, `real_time_horizon`), number of price scenarios, and solver ([`bidder.py:224`](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/bidder.py:224)).
- Price forecasts from forecaster (DA and RT).
- Realized DA prices/dispatches when computing RT bids.

**How it gets this information**

- Plugin creates `Bidder(...)` and passes model + forecaster + solver.
- DA prices/RT prices are pulled from forecaster in `compute_day_ahead_bids` ([`bidder.py:523`](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/bidder.py:523)).
- RT forecast comes from `forecast_real_time_prices`; realized DA prices/dispatches are passed by coordinator before RT solve ([`bidder.py:545`](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/bidder.py:545), [`bidder.py:570`](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/bidder.py:570)).

**What it does with the information**

It builds a scenario-based multiperiod optimization:
- One Pyomo block per scenario (`model.fs[i]`) ([`bidder.py:291`](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/bidder.py:291)).
- Calls your plant model `populate_model` on each scenario block ([`bidder.py:293`](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/bidder.py:293)).
- Adds market variables `day_ahead_power`, `real_time_underbid_power` and price parameters.
- Adds bidding constraints that enforce monotonic bid behavior across scenarios ([`bidder.py:1047`](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/bidder.py:1047), [`bidder.py:1080`](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/bidder.py:1080)).

**What optimization it solves**

It solves a **profit-maximization** problem:
- Objective sense is maximize ([`bidder.py:431`](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/bidder.py:431)).
- Revenue terms: DA settlement + RT deviation settlement.
- Cost term: subtract plant operating cost (`total_cost` from model object).
- Penalty term: subtract RT underbid penalty.

So conceptually:

$$\max \sum_{s,t}\left(\lambda^{DA}_{s,t}P^{DA}_{s,t}+\lambda^{RT}_{s,t}(P_{s,t}-P^{DA}_{s,t})-C_{s,t}-\pi^{under}u_{s,t}\right)
$$

subject to:
- Plant physics constraints (from your model’s `populate_model`).
- Bid monotonicity/non-anticipativity constraints across scenarios.
- DA upper-bound relaxation structure (`power_output + underbid >= day_ahead_power`) ([`bidder.py:396`](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/bidder.py:396)).

**What it outputs**

It outputs a Python dict of bids indexed by hour and generator:
- `p_cost` curve (piecewise points),
- `p_min`, `p_max`, `p_min_agc`, `p_max_agc`,
- startup/shutdown capacity, optional commitment.
See assembly in [`bidder.py:1216`](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/bidder.py:1216).

So output is not “money” directly; it is a **bid offer package** that market can clear.

**Who it passes output to**

- Bidder returns bids to `DoubleLoopCoordinator`.
- Coordinator injects them into Prescient RUC/SCED generator records:
  - DA: [`coordinator.py:501`](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/coordinator.py:501) -> [`coordinator.py:508`](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/coordinator.py:508)
  - RT: [`coordinator.py:636`](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/coordinator.py:636) -> [`coordinator.py:612`](/Users/vardhans/Projects/idaes-pse/idaes/apps/grid_integration/coordinator.py:612)


**Different Bidder classes in bidder.py**

| Class                     | Specialty                                                                          | Solves Optimization? | Typical Use                                                                            |
| ------------------------- | ---------------------------------------------------------------------------------- | -------------------- | -------------------------------------------------------------------------------------- |
| `AbstractBidder`          | Interface/base contract only                                                       | No                   | Base class for custom bidders                                                          |
| `StochasticProgramBidder` | Core stochastic multiperiod bidding framework (scenarios, objective, shared logic) | Yes                  | Parent for optimization-based bidders                                                  |
| `Bidder`                  | Builds offer curves from stochastic optimization; DA/RT market bidding             | Yes                  | Standard price-taker offer strategy for generator-style participants                   |
| `SelfScheduler`           | Enforces same power schedule across scenarios (self-scheduling behavior)           | Yes                  | Cases where you want deterministic schedule commitment rather than flexible bid curves |
| `ParametrizedBidder`      | Rule/parameter-based bids, lightweight logic                                       | No                   | Fast studies, heuristic policies, no solver dependency                                 |
| `PEMParametrizedBidder`   | Specialized parametrized bidder for PEM-type applications                          | No                   | PEM/hydrogen-specific workflows                                                        |



---

This is a great breakdown. To maximize the **signal-to-noise ratio**, we should strip away the "meta-talk" and focus strictly on the **functional inputs, logic, and outputs**.

Think of this as the "System Specification" for the Bidder agent.

---

### **Bidder: The Planning Agent**

**Core Role** The Bidder translates **internal asset physics** and **external price uncertainty** into a **market-compatible offer curve**. It ensures that any cleared dispatch is both profitable and physically feasible.

### **1. Input Vectors (The Observations)**

- **Asset Interface:** A model object (e.g., `ThermalGenerator`) providing the physics (`populate_model`), market variables (`power_output`), and internal economics (`total_cost`).
    
- **Price Scenarios:** Forecasted Day-Ahead (DA) and Real-Time (RT) price trajectories from the `Forecaster`.
    
- **Market State:** Realized DA prices and cleared dispatch (required for RT bidding).
    
- **Execution Parameters:** Optimization horizons, scenario counts, and solver configurations.
    

### **2. Internal Logic (The Computation)**

The Bidder executes a **Scenario-Based Stochastic Optimization**:

- **Temporal Expansion:** Uses `multiperiod.py` to clone the asset model across the planning horizon.
    
- **Stochastic Coupling:** Creates one model instance per price scenario.
    
- **Bidding Constraints:** Applies "shape rules" (e.g., monotonicity) to ensure the generated offer curves are logically consistent and valid for market submission.
    

### **3. Objective Function (The Goal)**

**Maximize Expected Profit:**

$$\mathbb{E}[\text{Market Revenue (DA + RT)}] - \text{Internal Operating Costs} - \text{Penalties (e.g., Under-delivery)}$$

The Bidder finds the optimal offer curve that balances potential revenue against the risk of violating the asset's physical limits.

### **4. Output (The Decision)**

A structured dictionary (JSON-compatible) for the Coordinator containing:

- **`p_cost`**: Piecewise linear price-quantity curves per hour.
    
- **Operational Bounds**: Dynamic $P_{min}$ and $P_{max}$ limits based on current asset state.
    

### **5. Integration Path (The Interface)**

1. **Bidder $\rightarrow$ Coordinator:** Returns the bid package.
    
2. **Coordinator $\rightarrow$ Prescient:** Injects bids into the Market UC/SCED.
    
3. **Prescient $\rightarrow$ Coordinator:** Returns cleared dispatch/prices to update the agent's state.
    

---

### **Summary Table: Signal vs. Noise**

|**Concept**|**The Signal (What matters)**|
|---|---|
|**Identity**|A stochastic optimizer that creates market bids.|
|**Dependency**|Relies on the `model_object` API (`populate_model`, `power_output`).|
|**Output**|Price-Quantity curves (`p_cost`) formatted for Prescient.|
|**Success**|Max profit while maintaining physical feasibility.|

    

---
