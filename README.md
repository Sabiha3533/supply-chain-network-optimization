# supply-chain-network-optimization
Prescriptive analytics and data governance case study optimizing multi-tier depot routing and constrained supplier blending, complete with data dictionaries and executive memos.
# Multi-Tier Supply Chain & Constrained Procurement Optimization

## Overview
Large-scale industrial and maritime sustainment programs require moving critical materials from multiple suppliers through regional transfer depots to end operational sites—while meeting strict technical specifications, carrier capacity limits, and cost targets.

This repository presents an **end-to-end prescriptive analytics and data governance case study** solving two interconnected logistics problems:
1. **Part 1 — Multi-Tier Transshipment & Distribution Network:** Minimizing total freight costs across a 3-echelon network (**2 Regional Supply Hubs → 4 Intermediate Transfer Depots → 6 End Operational Sites**), allowing both depot-routed and direct hub-to-site shipments.
2. **Part 2 — Constrained Multi-Vendor Procurement & Spec Blending:** Selecting optimal contract awards across **8 competing suppliers** to fulfill a **1,225,000-ton** annual demand target while enforcing technical quality thresholds ($\ge 19.0\%$ burnable volatility), labor continuity hedges ($\ge 50\%$ union-certified sourcing), and modal receiving caps (**Rail $\le 650\text{k}$ tons**; **Truck $\le 720\text{k}$ tons**).

---

## Relevance to Naval Sustainment & Data Management
* **Data Governance & Traceability:** Enforces standardized schemas, unit reconciliation (`kilotons` vs. `tons`), explicit masking of infeasible transport lanes, and pre-solve validation rules documented in [`docs/data_dictionary.md`](docs/data_dictionary.md).
* **Spares, Provisioning & Depot Logistics:** Directly mirrors Integrated Logistics Support (ILS) and crewing/supply workflows where assets move from vendors through intermediate depots to terminal facilities under strict capacity and compliance constraints.
* **Executive Decision Support:** Translates linear programming outputs—including **binding constraints** and **bottleneck trade-offs**—into a plain-English recommendation memo ([`docs/executive_decision_memo.md`](docs/executive_decision_memo.md)).

---

## Part 1: Multi-Tier Distribution Network Results

### Network Architecture & Constraints
* **Supply Hubs (2):** Hub Alpha (`300,000 tons` capacity) and Hub Beta (`400,000 tons` capacity).
* **Intermediate Transfer Depots (4):** Depot North (`140,000 tons`), Depot Central (`100,000 tons`), Depot East (`200,000 tons`), and Depot South (`80,000 tons`).
* **End Sites (6):** Sites S1 through S6 with fixed demand totaling **`430,000 tons`**.
* **Conservation & Capacity Rules:** Outbound hub volume $\le$ hub supply; inbound depot flow $=$ outbound depot flow $\le$ depot throughput; inbound site flow $=$ exact site demand.

### Optimal Routing Plan (Minimum Total Cost: **$541,000**)
Solved to global optimality in 7 simplex iterations (`0.02s`), achieving an average logistics cost of **$1.26/ton** across `430,000 tons`:

| Origin | Destination | Shipment Type | Optimal Volume (Tons) | Unit Cost ($/Ton) | Lane Cost ($) |
| :--- | :--- | :--- | :---: | :---: | :---: |
| Hub Alpha | Site S1 | Direct | 100,000 | $1.20 | $120,000 |
| Hub Alpha | Site S6 | Direct | 40,000 | $1.20 | $48,000 |
| Hub Beta | Depot Central | Hub → Depot | 100,000 | $0.50 | $50,000 |
| Hub Beta | Depot East | Hub → Depot | 110,000 | $0.70 | $77,000 |
| Hub Beta | Depot South | Hub → Depot | 80,000 | $0.40 | $32,000 |
| Depot Central | Site S2 | Depot → Site | 20,000 | $0.70 | $14,000 |
| Depot Central | Site S4 | Depot → Site | 70,000 | $1.20 | $84,000 |
| Depot Central | Site S5 | Depot → Site | 10,000 | $0.70 | $7,000 |
| Depot East | Site S5 | Depot → Site | 110,000 | $0.70 | $77,000 |
| Depot South | Site S3 | Depot → Site | 80,000 | $0.40 | $32,000 |
| **Total Network** | **All Active Lanes** | **Mixed Direct & Depot** | **430,000 (Delivered)** | **$1.26 (Avg)** | **$541,000** |

**Key Operational Insights:**
* **Hybrid Routing Efficiency:** Sites S1 and S6 are served directly from Hub Alpha (`140,000 tons`), bypassing depot handling fees where direct lanes are cost-competitive.
* **Binding Depot Bottlenecks:** **Depot Central (`100,000/100,000 tons`)** and **Depot South (`80,000/80,000 tons`)** operate at **100% capacity**, whereas Depot North (`0%`) remains idle due to higher downstream lane costs.

---

## Part 2: Constrained Multi-Vendor Procurement & Blending Results

### Problem Structure
Rather than disqualifying low-cost vendors whose material falls below the **19.0% minimum technical volatility threshold**, the linear optimization model blends lower-volatility, low-cost bids with higher-volatility supply to meet all operational and risk constraints simultaneously.

### Optimal Contract Allocation (Minimum Total Cost: **$73,267,500**)

| Vendor | Unit Bid ($/Ton) | Volatility (%) | Mode | Labor Status | Max Bid (kilotons) | Optimal Award (kilotons) | Total Spend ($) |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| Vendor A | $49.50 | 15% | Rail | Union | 300 | **55.0** | $2,722,500 |
| Vendor B | $50.00 | 16% | Truck | Union | 600 | **600.0** | $30,000,000 |
| Vendor C | $61.00 | 18% | Rail | Non-Union | 510 | **0.0** | $0 |
| Vendor D | $63.50 | 20% | Truck | Union | 655 | **20.0** | $1,270,000 |
| Vendor E | $66.50 | 21% | Truck | Non-Union | 575 | **100.0** | $6,650,000 |
| Vendor F | $71.00 | 22% | Truck | Union | 680 | **0.0** | $0 |
| Vendor G | $72.50 | 23% | Rail | Non-Union | 450 | **450.0** | $32,625,000 |
| Vendor H | $80.00 | 25% | Rail | Non-Union | 490 | **0.0** | $0 |
| **Total / Weighted** | **$59.81/ton** | **19.00%** | **505 Rail / 720 Truck** | **55.10% Union** | — | **1,225.0** | **$73,267,500** |

**Constraint Verification & Binding Bottlenecks:**
* **Technical Quality Spec:** Achieved **19.00% exact weighted volatility** (Binding Constraint).
* **Truck Receiving Infrastructure:** Utilized **720.0 / 720.0 kilotons (100%)** (Binding Constraint — expanding truck unloading capacity would unlock additional savings from low-cost truck vendors).
* **Rail Receiving Infrastructure:** Utilized **505.0 / 650.0 kilotons (77.7%)** (145.0 kilotons slack).
* **Union Sourcing Hedge:** Achieved **55.10% union supply** (`675.0 / 1,225.0 kilotons`), exceeding the `50.0%` floor with a `5.10%` safety buffer.

---

## Repository Structure
* `README.md` — Executive overview, mathematical summary, and verified optimization tables.
* `docs/data_dictionary.md` — Field definitions, data types, unit conversions, and QA validation rules.
* `docs/executive_decision_memo.md` — Non-technical briefing for operations and procurement leadership.

## Tools & Methodology
* **Optimization & Modeling:** Linear Programming (LP), Transshipment Network Flow, Blending Formulations (`Python / Gurobipy`, `SciPy`, `Excel Solver`)
* **Data Quality & Governance:** Schema Standardization, Feasibility Pre-Checks, Constraint Auditing, Sensitivity Analysis
