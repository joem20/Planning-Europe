# Planning Europe My Way

**A Graph-Based Route Planning Prototype for Selected European Cities**

## Project Overview

This project develops a graph-based route planning prototype for selected European cities. It investigates how transport connections can be represented as a graph and how shortest-path algorithms can identify cost-effective journeys.

The system considers two main optimisation criteria:

- Ticket price
- Journey time

A generalised cost function balances these criteria using a configurable value of time (λ).

## Research Question

How effectively can a graph-based multi-criteria optimisation system identify cost-effective multi-city travel routes across a defined European transport network, balancing ticket price and journey time?

## Dataset

The initial dataset (Dataset v1) contains:

- 10 European cities
- 27 direct rail and coach links
- One adult traveller
- Thursday 3 December 2026
- Departures between 08:00 and 20:00

The dataset was collected manually from transport operator websites on 5–6 November 2026.

Original fares are retained alongside their GBP conversions. The following fixed exchange rates were used:

- EUR: 0.834 GBP
- CHF: 0.9115 GBP

Source: fxfeed.io, 6 November 2026.

## Methodology

The project uses:

- Graph-based transport network modelling using an undirected weighted multigraph.
- Dijkstra's shortest-path algorithm.
- Generalised cost function:

  C(e) = P(e) + λT(e)

- A non-negative value of time (λ ≥ 0), tested across a range of £0–£30 per hour, with £5, £10 and £20 as the main evaluation scenarios.
- An expanded routing state represented as (city, incoming service), with individual services identified by their link ID.
- A fixed interchange allowance of 30 minutes per change of service, costed as λ × 0.5 hours.
- Sensitivity analysis to investigate how different values of λ affect recommended routes.
- Comparison against NetworkX to validate the routing implementation.

## Dataset Limitations and Modelling Assumptions

- Fares are indicative snapshots collected on 5–6 November 2026. Some fares are promotional and may change.
- Each link was researched in one direction only but is modelled as undirected, assuming identical journey times and fares in both directions. This reverse-direction assumption has not been independently verified.
- The dataset covers one travel date (3 December 2026) and one adult traveller, without modelling demand variation.
- The fixed 30-minute interchange allowance is a modelling assumption. Actual transfer times may differ, particularly where stations are separate.

## Project Scope

This is a deterministic optimisation prototype. It does not currently use machine learning, training data or a trained predictive model.

Weather suitability is considered a possible future extension.

## Technologies

- Python
- Google Colab
- Pandas
- NetworkX
- Matplotlib
- GitHub

## Dataset Versioning

Dataset v1 is maintained as a fixed baseline for reproducible analysis. Its collection methodology, selection rules, modelling assumptions and limitations are documented as part of the project.
