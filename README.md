# GTFS-RTP — Rich Travel Products Extension

**Version:** 0.1.0-draft  
**Author:** Peter Myers  
**Status:** Public Draft — RFC  
**License:** Creative Commons Attribution 4.0 (CC BY 4.0)

---

## What Is This?

GTFS (General Transit Feed Specification) is the global open standard for public transit data. It is consumed by Google Maps, Apple Maps, and thousands of transit agencies worldwide.

GTFS tells you **where** the bus stops and **when** it arrives. It says nothing about what you can **buy**, what experience you're entitled to, or how a tourism operator bundles a hop-on-hop-off route with a winery visit.

**GTFS-RTP (Rich Travel Products)** is a proposed open extension that fills that gap.

It adds a commerce and experience layer on top of GTFS — standardising how tourism transit operators define products, packages, entitlements, and commerce-enabled stops.

---

## The Problem

Tourism and transit are converging. Hop-on-hop-off buses, scenic rail journeys, experience-bundled transfers — these operators are building bespoke ticketing and entitlement systems because no open standard exists for their use case.

The result:

- No interoperability between operators and resellers
- No standard format for tour booking platforms to ingest tourism routes
- No way for Google/Apple Maps to understand that a stop has a sellable day pass
- Every operator invents the same wheel

GTFS-RTP proposes to fix this.

---

## What It Covers

| Layer | GTFS-RTP Table | Description |
|---|---|---|
| Routes | `rtp_routes` | Tourism route classes: HOHO, scenic, connector, region |
| Stops | `rtp_stop_context` | Commerce flags per stop: NFC, QR, sales enabled, stop role |
| Products | `rtp_products` | Sellable items: day pass, single ride, region pass |
| Packages | `rtp_packages` | Bundled experiences: transport + attraction clusters |
| Entitlements | `rtp_entitlements` | Access tokens: REGION, ROUTE, or TRIP scope |
| Trips | `rtp_trip_instances` | Scheduled departure instances |
| Capacity | `rtp_capacity_snapshot` | Real-time capacity per trip |
| Attribution | `rtp_attribution_points` | Commission and partner attribution |
| Events | `rtp_event_types` | Standardised telemetry event definitions |
| Vehicles | `rtp_vehicles` | Vehicle registry |
| Assignment | `rtp_trip_vehicle_assignment` | Vehicle-to-trip linkage |

All tables are designed to sit alongside standard GTFS files. Adoption is additive — existing GTFS consumers are unaffected.

---

## Design Principles

1. **Additive, not breaking.** GTFS-RTP files are supplementary. Any existing GTFS feed remains valid.
2. **CSV-first.** Matching GTFS conventions — plain CSV, human-readable, tooling-friendly.
3. **Commerce-aware.** First-class support for pricing, validity rules, and entitlement scopes.
4. **Channel-neutral.** Products declare allowed sales channels (WEB, APP, NFC, AGENT, PARTNER, WHOLESALE) — not the channel implementation.
5. **Tourism-native.** Built for hop-on-hop-off, scenic routes, experience bundles, and destination access products — not adapted from commuter transit.

---

## Quick Example

A tourist purchases a **Tamborine Mountain Day Bundle** from Brisbane. In GTFS-RTP:

**`rtp_routes.csv`** declares the route as `HOHO` class, market `BNE`.

**`rtp_stop_context.csv`** marks The Star Brisbane as a `HUB` stop with QR and sales enabled.

**`rtp_products.csv`** defines the `SINGLE_RIDE` connector ($49 AUD) and the `DAY_PASS` ($89 AUD).

**`rtp_packages.csv`** bundles both into one purchasable unit, with included attraction clusters.

**`rtp_entitlements.csv`** records the issued access token: ROUTE scope for the connector, REGION scope for the day pass.

Every step is machine-readable, interoperable, and open.

---

## Repository Structure

```
gtfs-rtp/
├── README.md               ← This file
├── SPEC.md                 ← Full field-by-field specification
├── CHANGELOG.md            ← Version history
├── schema/
│   ├── rtp_routes.json         ← JSON Schema
│   ├── rtp_stop_context.json
│   ├── rtp_products.json
│   ├── rtp_packages.json
│   ├── rtp_entitlements.json
│   ├── rtp_trip_instances.json
│   ├── rtp_capacity_snapshot.json
│   ├── rtp_attribution_points.json
│   ├── rtp_event_types.json
│   ├── rtp_vehicles.json
│   └── rtp_trip_vehicle_assignment.json
└── examples/
    └── tamborine-mountain/     ← Reference implementation (real-world data)
```

---

## Status

This is a **public draft**. The schema is derived from a live production implementation (JRNY Transit, Australia) and is being published to invite feedback, adoption, and co-authorship from the global transit and tourism community.

**We are actively seeking:**
- Transit agencies operating tourism or HOHO services
- Destination management organisations
- Tourism booking platforms
- GTFS tooling maintainers
- Researchers in MaaS (Mobility as a Service)

---

## Contributing

Open an issue. Start a discussion. Submit a PR.

If you operate a tourism transit service and want your use case reflected in the spec, we want to hear from you.

---

## Author

**Peter Myers**  
Founder, JRNY Transit  
GitHub: [@pmyers-abundance](https://github.com/pmyers-abundance)

---

## License

Creative Commons Attribution 4.0 International (CC BY 4.0)  
You are free to use, share, and adapt this specification with attribution.
