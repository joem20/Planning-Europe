
# FYP Decision Log

| ID | Date | Decision | Reason | Evidence |
|---|---|---|---|---|
| D01 | 5–6 Nov 2026 | Hand-built dataset: direct rail and coach links between 10 cities, Thu 3 Dec 2026, one adult, departures 08:00–20:00. | Supervisor asked for a documented dataset before routing; a single date keeps fares comparable. | `routes_dataset_v1.csv`, tag `dataset-v1` |
| D02 | 5–6 Nov 2026 | Selection rule: for each city pair, take the cheapest direct service within 10 minutes (inclusive) of the fastest in the window. Ties: earliest departure, then shorter journey. | A pure cheapest-fare rule picks slow services and removes the time/cost trade-off. | Dataset decisions note |
| D03 | 6 Nov 2026 | Original currency kept; converted to GBP at one fixed rate (1 EUR = 0.834 GBP, 1 CHF = 0.9115 GBP, fxfeed.io, 6 Nov 2026). | Consistent comparison; avoids rates changing between rows. | Dataset, README |
| D04 | 6 Nov 2026 | Berlin–Cologne uses FlixTrain (rail) because no direct coach exists on the date. | Only direct services are included. | Link L21 |
| D05 | 6 Nov 2026 | Cologne uses Köln Hbf only; Paris–Lyon uses Lyon Part-Dieu (Saint-Exupéry services excluded). | Station choice changes times and fares; one station per city keeps links comparable. | Dataset notes |
| D06 | 6 Nov 2026 | Corrected L06 operator to OUIGO Grande Vitesse (€25 fare). | The €25 fare belongs to OUIGO, not TGV INOUI. 
| D07 | 9 Nov 2026 | Links are modelled as undirected (same time and price both ways), although each was collected in one direction only. A reverse-direction collection was not done. | Fares changed after 5–6 Nov, so later reverse data would mix dates; a same-day check was skipped. Stated as a limitation. | Dataset decisions note (Limitations) |
| D08 | 11 Nov 2026 | Supervisor feedback: state λ ≥ 0, give the λ range and interchange allowance with justification, and make the routing state identify the incoming service. Adopted: λ from £0 to £30 per hour (main scenarios £5, £10, £20); 30-minute interchange allowance costed at λ × 0.5 h; state = (city, incoming service as link_id). | Dijkstra needs non-negative costs; mode alone cannot distinguish two rail services. | Supervisor email, 11 Nov 2026 |
| D09 | 11 Nov 2026 | Replied to the supervisor, noting that Dataset v1 was already collected and asking him to review the selection rule. | He was not aware of the dataset. | Email |
| D10 | 14 Nov 2026 | The project is described as a deterministic, graph-based optimisation prototype, not “AI-driven”; no machine learning is used. | Wording must match what is built. Title already says “graph-based”. | README, project title |
| D11 | 14 Nov 2026 | Created public GitHub repo `joem20/Planning-Europe`; froze Dataset v1 as tag `dataset-v1` (commit `7a13a8c`); notebook loads the CSV from the tag. | Reproducibility required by the brief. | Release page, notebook |
| D12 | 14 Nov 2026 | Built an undirected weighted MultiGraph containing 10 city nodes and 27 transport edges. Confirmed that the graph is connected. | A MultiGraph supports multiple transport options between the same city pairs. The connected network provides the foundation for route-finding algorithms. | Graph_Construction.ipynb |
| D13 | 16 Nov 2026 | Own Dijkstra validated against NetworkX (450 comparisons, no mismatches) | To check the reliability of my own implementation across five lambda values and all 90 ordered city pairs. | T03 results in Graph_Construction.ipynb; 450 matches, 0 mismatches. |
| D14 | 15 Nov 2026 | Negative λ is rejected with an error | Dijkstra needs non-negative costs; follows the supervisor's 11 Nov feedback (D08). | T08 in the test log |
| D15 | 17 Nov 2026 | λ tested at 0, 5, 10, 20 and 30 pounds per hour; λ = 0 treated as a limiting case | Time is free at 0, which produces long chains of coaches, so most analysis uses λ from 5. The range still needs a cited source. | T06 and T06b in the test log |
| D16 | 18 Nov 2026 | Symmetry test compares costs with a tolerance of 0.01 + 1e-9 | The first run flagged one pair because a one-penny rounding difference exceeded 0.01 in floating point. The failed run is kept in the log. | T06d in the test log |
| D17 | 19 Nov 2026 | Weather not pursued; recorded as future work | It was part of my initial idea (whole of Europe, cost, journey time and weather). At our first meeting, the supervisor advised that this was too big, and it was dropped before my first proposal. The ten cities also have broadly similar winter weather. | Supervision log |
| D18 | 19 Nov 2026 | Supervisor accepted the 30-minute interchange allowance (confirms D08) | The expanded model with state (city, incoming service) goes ahead. It simplifies the pure transfer penalty. | Supervision log, 19 Nov |
| D19 | 19 Nov 2026 | Links stay undirected (confirms D07); no full reverse-direction dataset | Supervisor accepted the limitation. A full reverse collection would repeat the whole dataset, and fares change over time. An optional sample check, both directions in one session, is planned if time allows. | Supervision log, 19 Nov |
| D20 | 19 Nov 2026 | Research gaps and the state of the art, using AI-assisted search with sources checked by me | Supervisor advised it on 19 Nov. I will check each source before using it, and I wrote the Dijkstra and test code myself. | Literature notes, AI-use entry |




- Web interface: Streamlit proposed; not yet confirmed.
- Source for the value-of-time range: to be found during the literature review.
- Whether weather is attempted: optional, decided later with supervisor.
