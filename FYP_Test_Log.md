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


