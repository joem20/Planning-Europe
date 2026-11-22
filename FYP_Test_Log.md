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
| T05b | `rebuild_route()` on an unreachable destination (city D) | Raises `ValueError` | 21 Nov 2026 | PASS |
| T06 | Lyon–Zurich lambda sensitivity: £0, £5, £10, £20 and £30 |Milan (L24, L15) at λ = 0, 5, 10; Paris (L06, L14) at 20, 30; all costs matched prediction.| 17 Nov 2026 | PASS |
| T06b | Full-network lambda sensitivity: 90 ordered city pairs across five lambda values | Record route, cost and winning service changes across 450 calculations | 17 Nov 2026 | PASS |
| T06c | Break-even analysis: Lyon to Zurich | Verify that my Dijkstra switches from the Milan route to the Paris route at the hand-calculated break-even (about £13.69/hour).| 20 Nov 2026 | PASS|
| T06d | Symmetry of route costs | All 45 unordered city pairs agree within £0.01 across five λ values | 18 Nov 2026 | PASS |
| T06e | Monotonicity of generalised costs | Minimum cost for every ordered city pair does not decrease as λ increases | 18 Nov 2026 | PASS |
| T07a | State-based Dijkstra on tiny test graph (A–B–C): interchange penalty applied at a service change, not at the origin | λ = 10: A→C direct, cost 84, links ['T3']. λ = 0: A→B→C, cost 40, links ['T1', 'T2']. A→A: cost 0, route ['A'], links [] | 21 Nov 2026 | PASS |
| T07 | State-based Dijkstra on Dataset v1: 30-minute interchange penalty (λ × 0.5 h) applied correctly at each service change | State-based result matches brute-force enumeration of simple paths with penalty λ × 0.5 × (links − 1), on a sample of city pairs and λ values | – | Not tested |
| T08 | Negative lambda validation | Negative λ raises ValueError | 15 Nov 2026 | PASS |
| T09 | Small graph: A–B–C–D | Minimum cost 8 via A→C→B→D | 15 Nov 2026 | PASS |
| T10 | Starting city equals destination | cost 0, route [start], no links, for London and Zurich | 20 Nov 2026 | PASS |
| T11 | Regression check: replace inline route-rebuilding loops in T02, T06 and T06b with `rebuild_route()` and rerun | Routes and costs identical to the earlier runs | 21 Nov 2026 | PASS |


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
- D is added as an isolated node with no connections.

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

**Date:** 17 November 2026

### Preliminary T06b findings – Full-network sensitivity

The full-network analysis completed 450 route calculations, covering 90 ordered city pairs across five lambda values (£0, £5, £10, £20 and £30).

A total of 66 out of 90 ordered journeys (73.3%) changed their selected transport links at least once. These represent 33 of the 45 unique city pairs.

The number of ordered journeys changing route between consecutive lambda values was:

| Lambda interval | Ordered journeys changing route |
|---|---:|
| £0 to £5 | 44 |
| £5 to £10 | 12 |
| £10 to £20 | 36 |
| £20 to £30 | 12 |

The most significant changes occurred between £0 and £5, followed by the range of £10 to £20. This suggests that the assigned value of journey time can significantly influence the routes chosen by the optimiser.

These findings are preliminary, and further analysis of individual route changes and transportation links is necessary.
### Prediction before testing – Transport-link frequency

**Date:** 17 November 2026

I expect the most frequently selected transport links to change as the value of time increases.

1. **At λ = £0:** I expect coach links, particularly FlixBus services L18, L19 and L20, to feature prominently in the top ten because ticket price is the only factor in the generalised cost.

2. **As λ increases:** I expect rail services to become more prominent and coach services to become less frequent in the selected routes, because journey time has an increasing influence on generalised cost.

3. **At λ = £30:** I expect faster rail connections, including Eurostar links L01, L03 and L04 and DB link L12, to feature more prominently. I also predict that Lyon–Milan rail link L25 may replace the slower coach link L24 in the most frequently selected links.

These are predictions, not established results. The frequency analysis will determine whether they are supported by the program output.
## T06b – Full-network lambda sensitivity analysis

**Date:** 17 November 2026

**Objective:** Examine how changing the value of time (λ) affects route selection across the complete ten-city transport network.

**Method:** The program tested five lambda values (£0, £5, £10, £20 and £30 per hour) across all 90 ordered pairs of distinct cities. This produced 450 route calculations. For each calculation, the selected route and its generalised cost were recorded.

### Prediction before testing

Before running the analysis, I predicted that coach links would dominate at λ = £0 because journey time has no cost at this value. I expected rail links to become more prominent as λ increased, because faster journeys would become more attractive.

I also predicted that L24 would be replaced by L25 in the top ten at λ = £30.

### Actual results

**Total route calculations:** 450

**Route changes between lambda values:**

| Lambda interval | Number of route changes |
|---|---:|
| £0 to £5 | 44 |
| £5 to £10 | 12 |
| £10 to £20 | 36 |
| £20 to £30 | 12 |

There were **66 ordered city pairs** with at least one route change across the lambda values, representing **33 unique unordered city pairs**.

### Coach and rail links in the top ten

| λ (£/hour) | Coach links | Rail links |
|---:|---:|---:|
| 0 | 6 | 4 |
| 5 | 4 | 6 |
| 10 | 3 | 7 |
| 20 | 2 | 8 |
| 30 | 1 | 9 |

The results show a clear increase in the proportion of rail links in the top ten as the value of time increases. Coach links decreased from six at λ = £0 to one at λ = £30.

### Most frequently selected links

The ten most frequently selected links at each lambda value were:

**λ = £0/hour**

| Link | Mode | Selections |
|---|---|---:|
| L19 | Coach | 38 |
| L18 | Coach | 36 |
| L20 | Coach | 26 |
| L27 | Rail | 22 |
| L23 | Coach | 22 |
| L06 | Rail | 20 |
| L16 | Coach | 18 |
| L24 | Coach | 14 |
| L15 | Rail | 12 |
| L10 | Rail | 10 |

**λ = £5/hour**

| Link | Mode | Selections |
|---|---|---:|
| L18 | Coach | 30 |
| L09 | Rail | 24 |
| L06 | Rail | 22 |
| L11 | Rail | 20 |
| L23 | Coach | 16 |
| L01 | Rail | 14 |
| L24 | Coach | 12 |
| L15 | Rail | 10 |
| L19 | Coach | 8 |
| L21 | Rail | 8 |

**λ = £10/hour**

| Link | Mode | Selections |
|---|---|---:|
| L09 | Rail | 26 |
| L18 | Coach | 26 |
| L06 | Rail | 22 |
| L11 | Rail | 20 |
| L23 | Coach | 16 |
| L01 | Rail | 16 |
| L21 | Rail | 12 |
| L24 | Coach | 12 |
| L15 | Rail | 10 |
| L10 | Rail | 8 |

**λ = £20/hour**

| Link | Mode | Selections |
|---|---|---:|
| L11 | Rail | 28 |
| L06 | Rail | 24 |
| L09 | Rail | 24 |
| L04 | Rail | 20 |
| L27 | Rail | 16 |
| L23 | Coach | 14 |
| L24 | Coach | 10 |
| L02 | Rail | 8 |
| L15 | Rail | 8 |
| L14 | Rail | 8 |

**λ = £30/hour**

| Link | Mode | Selections |
|---|---|---:|
| L11 | Rail | 26 |
| L06 | Rail | 24 |
| L09 | Rail | 24 |
| L04 | Rail | 20 |
| L27 | Rail | 16 |
| L23 | Coach | 12 |
| L25 | Rail | 10 |
| L14 | Rail | 10 |
| L05 | Rail | 8 |
| L02 | Rail | 8 |

These counts represent link uses across 90 ordered journeys at each lambda value. A link may be selected frequently because of its position and importance within the network; frequency does not necessarily mean that it is independently the most attractive service.

### Additional observations

- **Amsterdam–Berlin:** Three different route regimes appeared. L20 was selected at λ = £0 and £5. At λ = £10, the preferred route changed to Amsterdam → Cologne → Berlin using L10 and L21. At λ = £20 and £30, L12 became the preferred route. This demonstrates that the complete network can identify an intermediate route that would be missed by comparing direct links alone.
- **L01:** Appeared in the top ten at λ = £5 (14 selections) and λ = £10 (16 selections). It was outside the top ten at λ = £20 and £30, but the top-12 check showed it ranked 11th at £20 and 12th at £30.
- **L03:** Did not appear in either the top ten or the top twelve at λ = £20 or £30.
- **L12:** Did not appear in any of the five top-ten lists.
- **L24 and L25:** L24 appeared 10 times at λ = £20, while L25 appeared 10 times at λ = £30.

### Interpretation and limitations

The results support the prediction that increasing the value of time generally makes rail links more prominent. However, link frequency also reflects network position and the number of journeys that use a particular connection.

At λ = £0, journey time has no value, and there is no interchange penalty in the current basic model. This can produce long journeys involving several coach services. This provides a reason to investigate the expanded routing model and interchange penalty in T07.

### Result

**Result: PASS.** The full-network sensitivity analysis completed all 450 route calculations. The results showed changes in route selection across the five lambda values and identified differences in the frequency of selected transport links.

**Evidence:**  Output from the T06b sensitivity analysis and link-frequency cells in Graph_Construction.ipynb. Notebook commit: 11e2beb.

**Date:** 18 November 2026

### T06d — Symmetry test

Expected outcome:
All 45 unordered city pairs will have matching generalised costs in both directions (A → B and B → A), within £0.01, across all five λ values (0, 5, 10, 20 and 30).

The graph is undirected, and each transport link has the same price and journey time in both directions. Therefore, the minimum generalised cost should also be the same in both directions. This test does not validate the assumption that real fares and times are equal in both directions; this remains a stated limitation.

### T06e — Monotonicity test

Expected outcome: 
For every ordered city pair, the minimum generalised cost will not decrease as λ increases from £0 to £30 per hour across the five tested scenarios.
The formula is:

**C = P + λ × T**

where:
- C = generalised cost (£)
- P = ticket price (£)
- λ = value of time (£ per hour)
- T = journey time (hours)

Journey time (T) is non-negative. Increasing λ cannot decrease the cost of an individual route. Consequently, the minimum cost across all available routes cannot decrease either.

### T06d — Symmetry test results
- Initial run: 225 comparisons, 1 flagged failure.
- Maximum difference: 0.01000000000000512.
- Cause: floating-point representation made a one-penny difference marginally exceed the 0.01 threshold.
- Corrected run: 225 comparisons, 0 failures.
- Correction: tolerance adjusted to 0.01 + 1e-9.
- Status: PASS.
  
### T06e — Monotonicity test results
- Comparisons: 360.
- Failures: 0.
- Status: PASS.
- Result: Minimum generalised cost did not decrease as λ increased across the five scenarios.
  
### T10 – Starting city equals destination

**Status:** PASS (20 Nov 2026)

**Purpose:** To verify that Dijkstra and `rebuild_route()` correctly handle a journey where the starting city and destination are the same.

**Prediction (written 20 Nov 2026, about 07:30, before the run):** cost 0, route containing only the starting city, no links, for London and Zurich.

**Test procedure:** Dijkstra was run for London → London and Zurich → Zurich. The generalised cost, the `previous` dictionary and the route rebuilt by `rebuild_route()` were checked with assertions.

**Expected result**

- Generalised cost is 0.
- The start city does not appear in `previous`.
- The reconstructed route contains only the starting city.
- The list of transport links is empty.

**Actual result**

| Test case | Cost | Route | Links |
| --- | --- | --- | --- |
| London → London | 0 | ['London'] | [] |
| Zurich → Zurich | 0 | ['Zurich'] | [] |

**Conclusion:** Both cases passed all four assertions. This confirms the system handles a journey with identical origin and destination.

**Evidence:** `Graph_Construction.ipynb`, notebook commit eb4c65a.
### T07a – State-based Dijkstra on tiny test graph (interchange penalty)

**Date written:** Fri 20 Nov 2026, 21:01 (before any code)

**Aim:** Check that the state-based Dijkstra (state = (city, incoming_link_id)) applies the penalty λ × 0.5 h on each service change, and not at the origin.

**Test graph (not Dataset v1):**
| Link | From → To | Price (£) | Time (h) |
|---|---|---|---|
| T1 | A → B | 20 | 2.0 |
| T2 | B → C | 20 | 2.0 |
| T3 | A → C | 42 | 4.2 |

**Prediction before testing (hand calculation, λ = 10):**
- A → B → C without penalty: 20 + 20 + 10 × 4.0 = 80
- Penalty for one change at B: 10 × 0.5 = 5
- A → B → C with penalty: 85
- A → C direct: 42 + 10 × 4.2 = 84
- Expected result: A → C direct wins (84 < 85). Without the penalty, A → B → C would win (80 < 84).

**Prediction (λ = 0):** the penalty is 0. A → B → C costs 40 and wins over direct (42). Links ['T1', 'T2'].

**Prediction (A → A):** returns (0, ['A'], []).

**Expected outcome:**
| Case | Cost | Cities | Links |
|---|---|---|---|
| λ = 10, A → C | 84 | A, C | ['T3'] |
| λ = 0, A → C | 40 | A, B, C | ['T1', 'T2'] |
| λ = 10, A → A | 0 | A | [] |

**Pre-check (Fri 20 Nov, 22:29):**

Test graph built and set_generalised_costs(G_test, 10) run. Link costs T1 = 40.0, T2 = 40.0, T3 = 84.0, matching hand calculation. The first run raised NameError because the graph-building cell had not been run in that session; I ran the cells in order, and it passed. (T07a itself is still Not tested.)

**Actual result (Sat 21 Nov 2026, 13:56):**
| Case | Cost | Cities | Links |
|---|---|---|---|
| λ = 10, A → C | 84.0 | A, C | ['T3'] |
| λ = 0, A → C | 40.0 | A, B, C | ['T1', 'T2'] |
| λ = 10, A → A | 0 | A | [] |

All three matched the predictions written before coding. `set_generalised_costs` was rerun with the matching λ before each case, because the function reads the stored generalised costs.

**Sanity check:** with the penalty set to 0 in a scratch copy, Test 1 returned A→B→C at 80.0, so the test detects a missing penalty.

**Status:** PASS (tiny graph only; Dataset v1 is covered by T07)

**Evidence:** Test log commit 
7ba26b2 code in notebook commit 1c73613
### T06c – Break-even analysis: Lyon to Zurich

**Date written:** Sat 21 Nov 2026 (predictions worked out about 15:23, before the test code)

**Aim:** Verify that my Dijkstra switches from the Milan route to the Paris route at the hand-calculated break-even (about £13.69/hour).

**Candidate routes (stored values):**
| Route | Links | Price (£) | Time (h) |
|---|---|---|---|
| Via Milan | L24 + L15 | 50.66 | 9.366 |
| Via Paris | L06 + L14 | 96.74 | 6.000 |

**Hand-calculated break-even:** λ* = (96.74 − 50.66) / (9.366 − 6.000) = 46.08 / 3.366 ≈ £13.68984 per hour.

**Prediction before testing:**
| λ | Milan cost (£) | Paris cost (£) | Expected winner | Expected links |
|---|---|---|---|---|
| λ* − 0.01 (13.67984) | 178.78538144 | 178.81904 | Milan | L24, L15 |
| λ* + 0.01 (13.69984) | 178.97270144 | 178.93904 | Paris | L06, L14 |
| λ* | equal | equal | no winner predicted | Dijkstra cost equals both route costs within 1e-9 |

The cost gap at 0.01 either side of λ* should be 3.366 × 0.01 = £0.03366 in each direction.

**Note on process:** my first-hand calculations contained two arithmetic slips (a wrong Milan price, 50.634 instead of 50.66, and an addition error). I found them by checking that the gaps on either side of λ* should be equal. The figures above are the corrected ones.

**At λ\*:** the result is not checked for which route wins, because the two costs are equal and the choice depends on tie-breaking and floating-point rounding.

**Actual result – first run (Sat 21 Nov 2026, 15:36):**
| λ (£/hour) | Dijkstra cost (£) | Cities | Links |
|---|---|---|---|
| 13.679840 (λ* − 0.01) | 178.78537743 | Lyon, Milan, Zurich | ['L24', 'L15'] |
| 13.699840 (λ* + 0.01) | 178.93903743 | Lyon, Paris, Zurich | ['L06', 'L14'] |
| 13.689840 (λ*) | 178.87903743 | Lyon, Paris, Zurich | ['L06', 'L14'] |

**Comparison with prediction:** The winning routes and links matched on both sides of the break-even: Milan below, Paris above. The costs differed from the hand-calculated figures by about 4.0 × 10⁻⁶ (Milan) and 2.6 × 10⁻⁶ (Paris). The hand calculation used the rounded break-even (13.68984), and the code used the unrounded value (about 13.6898396), so each λ in the code was lower by about 4.3 × 10⁻⁷; multiplied by each route's time, this accounts for the differences. This was a rounding difference in my prediction, not a fault in the code.

**Assertion run (Sat 21 Nov 2026, 15:47):** An assertion cell compared the Dijkstra output with expected values computed from the stored route totals and the unrounded break-even λ*, using a tolerance of 1e-9:
- λ* − 0.01: links ['L24', 'L15'] (Milan), cost matches the Milan formula.
- λ* + 0.01: links ['L06', 'L14'] (Paris), cost matches the Paris formula.
- λ*: Milan and Paris costs are equal, and the Dijkstra cost equals both.

All assertions passed.

**Sanity check (Sat 21 Nov 2026, 16:04):** The Milan-section assertion (below break-even) was deliberately changed to expect the Paris links ['L06', 'L14']. The cell raised an `AssertionError` on that line, so the test can fail. An earlier attempt had passed because the wrong line had been edited, and it was repeated correctly. The line was restored, and the clean rerun (16:06) passed: all assertions passed.

**Note on process:** my first-hand calculations contained two arithmetic slips (a wrong Milan price, 50.634 instead of 50.66, and an addition error). I found them by checking that the cost gaps on either side of λ* should be equal. The predictions above are the corrected figures, written before any test code was run.

**Status:** PASS

**Evidence** Notebook commits 6eff1f1 (assertions pass) and f34a985 (after the sanity check and clean rerun)
### T11 – Regression check: replace inline route-rebuilding loops with rebuild_route()

**Date written:** Sat 21 Nov 2026, 18:25 (before any code changes)

**Aim:** After replacing the inline route-rebuilding loops in T02, T06 and T06b with `rebuild_route()`, routes and costs are identical to the earlier runs.

**Baseline:** a deep copy of `sensitivity_results` (450 entries) taken before any change, plus the T02 and T06 results recorded in this log.

**Prediction before testing:**
| Test | Expected result after the change |
|---|---|
| T02 | Lyon → Milan → Zurich, links L24 + L15, cost £144.32 |
| T06 | Same five results as the original T06 test (table below) |
| T06b | 450 of 450 entries identical to the baseline (route links and cost to 2 dp): 0 differences |
| T06b link counts | Identical top-ten link counts at λ = 0, 5, 10, 20 and 30 |

**T06 expected results (Lyon to Zurich):**
| λ (£/hour) | Expected route | Links | Cost |
|---|---|---|---|
| 0 | Lyon → Milan → Zurich | L24 + L15 | £50.66 |
| 5 | Lyon → Milan → Zurich | L24 + L15 | £97.49 |
| 10 | Lyon → Milan → Zurich | L24 + L15 | £144.32 |
| 20 | Lyon → Paris → Zurich | L06 + L14 | £216.74 |
| 30 | Lyon → Paris → Zurich | L06 + L14 | £276.74 |

**Method:** new cells use `rebuild_route()` and write to new variables. Results are compared with the baseline entry by entry. The original cells are left unchanged until the comparison passes.

**Actual result (Sat 21 Nov 2026):**
| Check | Time | Result |
|---|---|---|
| Baseline saved: `sensitivity_results_original`, `results_lookup_original` | 18:37 | 450 entries each; first entry Amsterdam → Berlin, ('L20',), £24.49 |
| Reproducibility: original T06b loop rerun in the same session, compared with baseline | 19:39 | 450 / 450 entries, 0 differences |
| T06b refactored with `rebuild_route()`, compared with baseline (lambda, source, target, links, cost) | 19:58 | 450 / 450 entries, 0 differences; assertion passed |
| T06b top-ten link counts at λ = 0, 5, 10, 20, 30 | 20:14 | Identical at all five λ values |
| T02 with `rebuild_route()` | 20:20 | Lyon → Milan → Zurich, ['L24', 'L15'], £144.32 |
| T06 (Lyon–Zurich, five λ values) with `rebuild_route()` | 20:24 | All five results matched the prediction table |
| Structural validation of all 450 city sequences | 20:26 | 0 invalid |

All results matched the predictions written before any code was changed.

**Sanity checks:**
- The comparison's assertion was temporarily changed from `== 0` to `== 1` and raised `AssertionError`, which shows the assertion fires; it does not show the comparison can detect a difference.
- A deliberately corrupted city sequence in a scratch copy (first route's start city replaced with "WrongCity") was reported as 1 invalid sequence out of 450 by the structural validation check.
- The baseline comparison was not separately tested with a deliberately altered result, because the corrupted-city test does not exercise it (the comparison checks links and cost, not city sequences). This is a limitation of the evidence.

**Notes:** `rebuild_route()` was already defined directly after `dijkstra()`, so no cells were moved. The original T06b cell and its results were left unchanged, and the refactored version wrote to a separate list.

**Status:** PASS 

**Evidence:** Notebook commit 8da3e92; test log commit f398019
### T05b: Isolated city (unreachable destination)

**Prediction written (before running any code):** Sun 22 Nov 2026, 19:36

**Setup:** The original `dijkstra` (not `dijkstra_with_transfers`) is run on a scratch copy of `G_test` with an isolated node D added (no links). Dataset v1 is not touched.

**Prediction:**
1. `dijkstra(G_scratch, "A", "D")` returns the `distances` and `previous` dictionaries.
2. `distances["D"]` remains `float("inf")`, because D is unreachable.
3. `previous` is initialised as an empty dictionary and only receives entries when a cheaper route is found, so D is absent from `previous`.
4. Calling `rebuild_route(previous, "A", "D")` raises `ValueError` with the exact message `Destination is unreachable` (checked against the code in `rebuild_route`, no full stop).
5. The guard expected to trigger is `if current not in previous:`.

**Failure conditions:** `rebuild_route()` returns an empty or incorrect route, does not raise the expected `ValueError`, or raises an unrelated error.

**Can-it-fail check:** Deliberately use the incorrect expectation that no error is raised. The resulting failed assertion should show that the test detects the `ValueError`. This checks the test's ability to detect an unexpected outcome. It does not independently prove that the guard works correctly or that it would detect an incorrect route.

**Result:** PASS (Sun 22 Nov 2026, about 20:20)

**Setup notes (found while writing the test, not in the committed prediction):**
- First run of `dijkstra` raised `KeyError: 'generalised_cost'`, because `G_test` only stores `price_gbp` and `time_hours` and `dijkstra` reads the stored cost. This was a setup slip, not a test result. `set_generalised_costs(G_scratch, 10)` was added before `dijkstra`, with λ = 10. D has no links, so λ cannot change the expected outcome.
- The scratch graph was created with `G_test.copy()` and D added with `add_node("D")`. Checked: scratch cities A, B, C, D; 3 links; original `G_test` still A, B, C.

**Observed:**
1. `dijkstra(G_scratch, "A", "D")` returned both dictionaries without error.
2. `distances["D"]` = `inf`.
3. `"D" in previous_scratch` = `False`.
4. `rebuild_route(previous_scratch, "A", "D")` raised `ValueError` with the message `Destination is unreachable`.

**Prediction 5 (guard line):** consistent with the `if current not in previous` guard. The line that fired was not observed separately; it is inferred from the matching exception type and message.

**Can-it-fail Result:** The same call was run with the wrong expectation (no error raised). It failed with `AssertionError: Expected no error but got ValueError`, so the test can detect the `ValueError`. This does not prove the guard is correct, or that a wrong route would be detected.

**Limitation:** Tested on a 3-city scratch graph with one isolated city and the original `dijkstra` only. `dijkstra_with_transfers` and the real Dataset v1 are not covered by this test.

**Evidence:** Prediction commit 79f8d70; notebook commit ab16fb5 and test log commit fc22d2d

