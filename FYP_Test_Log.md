# FYP Test Log

This file records tests carried out during the development of the graph-based route planning system.

## Test Summary

| Test ID | Test description | Expected outcome | Date | Status |
|---|---|---|---|---|
| T01 | Dijkstra: London to Paris, λ = 10 | Eurostar L01, generalised cost £81.33 | 15 Nov 2026 | PASS |
| T02 | Multi-connection journey: Lyon to Zurich, λ = 10 | Lyon–Milan–Zurich using L24 and L15, total cost £144.32 | — | Not tested |
| T03 | Compare own Dijkstra with NetworkX | Same minimum generalised cost and route | — | Not tested |
| T04 | London to Paris, λ = 0 | FlixBus L16 selected, cost £28.49 | — | Not tested |
| T05 | Unreachable destination | Algorithm handles a destination with no available route | — | Not tested |
| T06 | Lambda sensitivity: £0, £5, £10, £20 and £30 | Record route, cost and winning service changes against break-even values | — | Not tested |
| T07 | Interchange penalty | Correct 30-minute penalty applied when changing services | — | Not tested |
| T08 | Negative lambda validation | Negative λ raises ValueError | — | Not tested |
| T09 | Small graph: A–B–C–D | Minimum cost 8 via A→C→B→D | — | Not tested |
| T10 | Starting city equals destination | Returns generalised cost 0 | — | Not tested |

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
