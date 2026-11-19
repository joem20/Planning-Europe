# FYP Supervision Log

| Date | Type | Key points | Decisions / feedback | Actions |
| --- | --- | --- | --- | --- |
| 6 Oct 2026 | First meeting  | My initial idea: a route planner covering the whole of Europe, balancing cost, journey time and weather. Discussed keeping an FYP development and decision log. | Supervisor advised the project was too big and the scope should be reduced. Weather and Europe-wide coverage were dropped before my first proposal. | Reduce scope; write and submit the proposal; start a decision log. |
| Fri 23 Oct 2026, 1:00 pm UK (Teams) | Meeting | Prototype demonstration; project scope, datasets and feedback on direction. | Approved to explore available datasets for the project.  | |
| 11 Nov 2026 |  email| Dataset and routing methodology. | Lambda must be non-negative, because Dijkstra needs non-negative edge costs. The lambda range and the interchange allowance both need justifying. The routing must account for the incoming service, using a state such as (city, incoming_link_id). | Add the negative-lambda check (T08). Research a source for the lambda range. Build the expanded interchange model. |
| Thu 19 Nov 2026 | Meeting (about 10 minutes) | Progress: dataset v1 frozen (27 links, 10 cities); own Dijkstra validated against NetworkX (450 of 450 match); lambda sensitivity analysis; interchange model; direction of links; literature gaps. | 30-minute interchange allowance accepted. Undirected links accepted, with the one-direction data collection stated as a limitation; return-direction data optional if time allows. Supervisor advised researching gaps and comparing with the state of the art. | Build the interchange model and test it (T07). Literature review: gaps and comparison with existing planners; find a source for the lambda range. Finish the route-rebuild function, T06c and T10. Update README and logs. |

## Notes
- 19 Nov: the meeting was short, so I emailed a follow-up summary on 19 Nov 2026.
- Weather is not pursued and is recorded as future work (see decision log).
- A full reverse-direction dataset was judged impractical; an optional sample check is planned if time allows.
