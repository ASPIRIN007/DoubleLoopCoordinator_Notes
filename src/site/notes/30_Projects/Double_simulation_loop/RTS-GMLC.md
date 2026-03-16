---
{"dg-publish":true,"permalink":"/30-projects/double-simulation-loop/rts-gmlc/","created":"2026-03-04T11:03:42.380-05:00","updated":"2026-03-16T13:44:54.225-04:00"}
---


---

# RTS-GMLC Dataset (Reliability Test System – Grid Modernization Lab Consortium)

The **RTS-GMLC dataset** is a synthetic power-system test case developed by the U.S. DOE Grid Modernization Lab Consortium. It is an updated version of the IEEE RTS-96 system designed for **modern grid studies involving renewable integration, unit commitment, and electricity market simulation**.

The dataset is widely used because it includes **network topology, generator data, and high-resolution time-series for load and renewables**.

---

# Network Structure

| Component          | Count   |
| ------------------ | ------- |
| Buses              | **73**  |
| Transmission lines | **105** |
| Transformers       | **15**  |
| DC lines           | **1**   |
| Total branches     | **121** |

The buses represent nodes where generation, demand, and transmission connect, enabling studies of **power flow and congestion (LMP formation)**.

---

# Generation Fleet

The system contains **156 generators** across multiple technologies.

|Type|Description|
|---|---|
|Thermal|Coal, natural gas combined cycle (CC), combustion turbines (CT), oil|
|Nuclear|Large baseload units|
|Hydro|Reservoir and run-of-river plants|
|Wind|Utility-scale wind farms|
|Solar PV|Utility PV + rooftop PV|
|Storage|Battery energy storage systems|

Total installed capacity is roughly **13–14 GW**, with peak demand around **11–12 GW**.

---

# Time-Series Data

RTS-GMLC provides **chronological hourly profiles** for:

- Load at each bus
- Wind generation
- Solar PV generation
- Hydro availability

These profiles are **weather-consistent** and allow realistic simulation of renewable variability.

---

# Role in IDAES + Prescient Workflow

In your project:

- **RTS-GMLC** → provides the **power system environment** (generators, loads, network).
- **Prescient** → runs **day-ahead UC and real-time SCED markets** on this dataset.
- **IDAES** → inserts asset models (e.g., IDC) that interact with the market.

Thus, RTS-GMLC defines the **grid context**, while IDAES models the **participating asset behavior**.