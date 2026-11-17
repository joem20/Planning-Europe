# FYP Test Log

This file records tests conducted during the development of the graph-based route planning system.

## Test Summary

| Test ID | Test description | Expected outcome | Date | Status |
|---|---|---|---|---|
| T01 | Dijkstra: London to Paris, λ = 10 | Eurostar L01, generalised cost £81.33 | 15 Nov 2026 | PASS |
| T02 | Multi-connection journey: Lyon to Zurich, λ = 10 | Minimum-cost route via L24 and L15, total cost £144.32 | 16 Nov 2026 | PASS |
| T03 | Full comparison of own Dijkstra with NetworkX: 5 lambda values and 90 ordered city pairs | 450/450 matches, 0 mismatches | 16 Nov 2026 | PASS |
| T04 | London to Paris, λ = 0 | FlixBus L16 selected, cost £28.49 | 15 Nov 2026 | PASS |
| T05 | Unreachable destination | D exists in graph but is disconnected; Dijkstra returns infinity | 15 Nov 2026 | PASS |
| T06 | Lyon–Zurich lambda sensitivity: £0, £5, £10, £20 and £30 |Milan (L24, L15) at λ = 0, 5, 10; Paris (L06, L14) at 20, 30; all costs matched prediction.| 17 Nov 2026 | PASS |
| T06b | Full-network lambda sensitivity: 90 ordered city pairs across five lambda values | Record route, cost and winning service changes across 450 calculations | - | In progress |
| T06c | Break-even analysis: Lyon to Zurich | Verify the route-switching point against the hand calculation of approximately £13.69/hour | - | Not tested |
| T07 | Interchange penalty | Correct 30-minute penalty applied when changing services | — | Not tested |
| T08 | Negative lambda validation | Negative λ raises ValueError | 15 Nov 2026 | PASS |
| T09 | Small graph: A–B–C–D | Minimum cost 8 via A→C→B→D | 15 Nov 2026 | PASS |
| T10 | Starting city equals destination | Returns generalised cost 0 | – | Not tested |


## T01 – Basic Dijkstra test: London to Paris

**Date:** 15 November 2026

**Component:**  Dijkstra implementation

**Objective:** Test whether the algorithm identifies the minimum generalised cost between London and Paris.

**Test inputs:**
- Starting city: London
- Destination city: Paris
- Lambda: £10 per hour
- Dataset: Dataset v1

**Expected result:** Direct Eurostar service L01 should have a generalised cost of £81.33.

**Actual result:**
- Shortest generalised cost: £81.33
- Previous city: London
- Previous link: L01

**Result:** PASS

**Evidence:** Output from the Dijkstra test cell in `Graph_Construction.ipynb`.

**Comments:** The algorithm successfully calculated the minimum generalised cost for this direct journey. Further testing is required for multi-connection journeys, parallel transport options and comparison against NetworkX.

## T09 – Small Graph: Known Shortest Route

**Date:** 15 November 2026

**Component:** Dijkstra implementation

**Objective:** Test the Dijkstra algorithm using a small graph with manually verifiable edge costs and a known shortest route.

**Test inputs:**

Starting node: A

Destination node: D

The test graph contains five connections:

| Connection | Generalised cost |
|---|---:|
| A–B | 4 |
| A–C | 2 |
| C–B | 1 |
| B–D | 5 |
| C–D | 8 |

**Expected result:**

The minimum-cost route should be A → C → B → D.

Total cost: 2 + 1 + 5 = **8**.

**Initial failed attempt and correction:**

The first test used `nx.Graph()` and `add_weighted_edges_from()`. This stored edge costs under the `weight` attribute, whereas the Dijkstra implementation expected each edge to have a `generalised_cost` attribute and used MultiGraph edge iteration.

This caused a `TypeError: 'int' object is not subscriptable` when the algorithm attempted to access the edge data.

The test graph was corrected by:

- Changing the test graph to `nx.MultiGraph()`.
- Adding edges individually with `generalised_cost` attributes.
- Assigning link IDs as edge keys.

This made the test graph compatible with the existing Dijkstra implementation.

**Actual result:**

- Shortest cost: 8
- Previous node and link for D: B, BD
- Reconstructed route: A → C → B → D

**Result:** PASS

**Evidence:** Output from the corrected test graph creation, Dijkstra execution and route reconstruction cells in `Graph_Construction.ipynb`. Commit `a1b5527`.
## T04 – London to Paris: Lambda = 0

**Date:** 15 November 2026

**Component:** Generalised cost calculation and Dijkstra implementation

**Objective:** Test whether the algorithm selects the cheapest transport service when journey time does not influence generalised cost.

**Test inputs:**

- Starting city: London
- Destination city: Paris
- Lambda: £0 per hour
- Dataset: Dataset v1

**Expected result:**

With λ = 0, generalised cost depends entirely on ticket price.

FlixBus link L16 should be selected at £28.49.

**Actual result:**

- Lambda: 0
- Shortest generalised cost: £28.49
- Previous city and link: London, L16
- Selected service: FlixBus

After the test, the graph's generalised costs were reset to λ = 10 for subsequent tests.

**Result:** PASS

**Evidence:** Output from the T04 Dijkstra test cell in `Graph_Construction.ipynb`. Commit `c25bf4f`.

## T05 – Unreachable destination

**Date:** 15 November 2026

**Component:**  Dijkstra implementation

**Objective:** Verify that Dijkstra handles a destination that exists in the graph but cannot be reached from the starting city.

**Test setup:**
- Starting node: A
- Destination node: D
- A–B cost: 4
- B–C cost: 3
- D added as an isolated node with no connections.

**Expected result:** Dijkstra should return infinity for D, without raising an error.

**Actual result:**
- Nodes: A, B, C, D
- Edges: A–B and B–C
- Shortest cost: infinity
- Is destination unreachable: True

**Result:** PASS

**Evidence:** Output from the T05 graph creation and Dijkstra execution cells in `Graph_Construction.ipynb`. Commit `f7d544b`.
## T08 – Negative lambda validation

**Date:** 15 November 2026

**Component:** Generalised cost calculation

**Objective:** Verify that the function rejects negative lambda values, following the requirement that λ must be non-negative.

**Test input:**
- Lambda: -1

**Expected result:** The function should raise a ValueError with the message "lambda must be non-negative".

**Actual result:**
- Negative lambda rejected.
- Error: lambda must be non-negative.
- Main graph costs reset to λ = 10 after the test.

**Result:** PASS

**Evidence:** Output from the T08 negative lambda validation cell in `Graph_Construction.ipynb`. Commit `5513669`.
## T02 – Multi-city journey: Lyon to Zurich (λ = £10/hour)

**Date:** 16 November 2026

**Objective:** Check that Dijkstra finds the minimum-cost multi-city route.

**Hand calculation (done before comparing with the program output):**
Generalised cost = price (£) + 10 × time (hours), from `routes_dataset_v1.csv`.

| Link | Price (£) | Time (h) | Cost |
|---|---:|---:|---:|
| Lyon–Milan FlixBus (L24) | 21.49 | 6.083 | 21.49 + 60.83 = 82.32 |
| Milan–Zurich (L15) | 29.17 | 3.283 | 29.17 + 32.83 = 62.00 |
| Lyon–Paris OUIGO (L06) | 20.85 | 1.933 | 20.85 + 19.33 = 40.18 |
| Paris–Zurich (L14) | 75.89 | 4.067 | 75.89 + 40.67 = 116.56 |

| Candidate route | Total cost |
|---|---:|
| Lyon → Milan → Zurich (L24, L15) | 144.32 |
| Lyon → Paris → Zurich (L06, L14) | 156.74 |

**Expected result:** Lyon → Milan → Zurich via L24 and L15, cost £144.32.

**Date:** 17 November 2026

**Prediction before testing – Lyon to Zurich**

Based on the hand calculations, I expect the route via Milan (L24 and L15) to be preferred at lower lambda values because it has a lower ticket price. At higher lambda values, I expect the route via Paris (L06 and L14) to become preferable because it has a shorter journey time.

| Lambda (£/hour) | Predicted route |
|---:|---|
| 0 | Lyon → Milan → Zurich (L24, L15) |
| 5 | Lyon → Milan → Zurich (L24, L15) |
| 10 | Lyon → Milan → Zurich (L24, L15) |
| 20 | Lyon → Paris → Zurich (L06, L14) |
| 30 | Lyon → Paris → Zurich (L06, L14) |

**Predicted break-even point:** Approximately £13.69 per hour, meaning I expect the preferred route to change between λ = £10 and λ = £20.

This prediction is based on comparing these two candidate routes. The program will determine the actual minimum-cost route across the complete network.
**Actual results – Lyon to Zurich**

The program produced the following results:

| Lambda (£/hour) | Actual route | Transport links | Generalised cost (£) |
|---:|---|---|---:|
| 0 | Lyon → Milan → Zurich | L24, L15 | 50.66 |
| 5 | Lyon → Milan → Zurich | L24, L15 | 97.49 |
| 10 | Lyon → Milan → Zurich | L24, L15 | 144.32 |
| 20 | Lyon → Paris → Zurich | L06, L14 | 216.74 |
| 30 | Lyon → Paris → Zurich | L06, L14 | 276.74 |

**Comparison with prediction:** All five actual route selections matched the predictions.

**Observed route change:** The preferred route changed from via Milan to via Paris between λ = £10 and λ = £20 per hour. This agrees with the predicted break-even point of approximately £13.69 per hour.

**Result:** The Lyon–Zurich sensitivity test matched the predictions.
**Evidence:** Prediction written and committed on 17 Nov 2026 (16:24) before the run; results recorded in commit `fcd7d2e`
