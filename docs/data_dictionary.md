# Data Dictionary & Quality Assurance (QA) Protocol

## 1. Distribution Network Schema (`Part 1`)

| Field Name | Table / Entity | Data Type | Unit | Validation Rule / Constraint | Description |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `node_id` | Locations | String (PK) | — | Non-null, unique identifier | Unique code for Supply Hubs, Depots, and Sites |
| `node_type` | Locations | Categorical | — | $\in \{\text{Hub}, \text{Depot}, \text{Site}\}$ | Functional role within the 3-tier network |
| `supply_cap` | Supply Hubs | Float | Tons | $> 0$; $\sum \text{supply} \ge \sum \text{demand}$ | Maximum outbound material capacity per hub |
| `throughput_cap` | Depots | Float | Tons | $> 0$ | Maximum transshipment volume handled by depot |
| `demand_req` | End Sites | Float | Tons | $> 0$ | Exact material tonnage required at terminal site |
| `unit_cost` | Cost Matrix | Float | $/Ton | $> 0$ or `NULL` (infeasible lane) | Freight cost per ton; `NULL` arcs excluded from solver |
| `flow_qty` | Solution Output | Float | Tons | $\ge 0$ | Optimal material volume routed along active arc |

## 2. Vendor Procurement Schema (`Part 2`)

| Field Name | Data Type | Unit | Validation Rule / Constraint | Description |
| :--- | :--- | :--- | :--- | :--- |
| `vendor_id` | String (PK) | — | Non-null, unique | Participating supplier identifier |
| `unit_price` | Float | $/Ton | $> 0$ | Delivered bid price per short/metric ton |
| `volatility_pct` | Float | Percentage | $0.0 \le v \le 100.0$; weighted avg $\ge 19.0\%$ | Burnable volatile matter specification |
| `transport_mode` | Categorical | — | $\in \{\text{Rail}, \text{Truck}\}$ | Inbound delivery mode subject to terminal caps |
| `union_certified` | Binary | Boolean | $\in \{0, 1\}$; weighted share $\ge 50.0\%$ | Labor continuity compliance indicator |
| `max_capacity` | Float | Kilotons | $> 0$ (`1 kiloton = 1,000 tons`) | Maximum annual contract volume offered by vendor |
| `award_qty` | Float | Kilotons | $0 \le \text{award\_qty} \le \text{max\_capacity}$ | Recommended annual contract award |

## 3. Pre-Solve Data Quality Controls
1. **Unit Reconciliation:** All procurement inputs standardized to `kilotons` (`1,000 tons`) prior to matrix construction to prevent scale mismatches.
2. **Infeasible Arc Masking:** Missing entries (`-`) in the transportation cost matrix are explicitly filtered out of the decision-variable index set rather than imputed as `0`.
3. **Mass-Balance Audit:** Automated post-solve checks verify `Inbound Flow - Outbound Flow = 0` at every intermediate depot to within `< 1e-6` numerical tolerance.
