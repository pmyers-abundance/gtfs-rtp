# GTFS-RTP Specification

**Version:** 0.1.0-draft  
**Author:** Peter Myers  
**Status:** Public Draft  

---

## 1. Overview

GTFS-RTP extends GTFS with eleven supplementary CSV files. Each file is optional and additive. Implementations may adopt any subset of tables.

All `rtp_*` identifiers use a namespaced prefix convention:

| Prefix | Entity |
|---|---|
| `RTP:` | Agency |
| `RTPR:` | Route |
| `RTPP:` | Product |
| `RTPK:` | Package |
| `RTPE:` | Entitlement |
| `RTPT:` | Trip Instance |
| `RTPV:` | Vehicle |

---

## 2. Tables

### 2.1 `rtp_routes.csv`

Tourism-class routes. Supplements GTFS `routes.txt`.

| Field | Type | Required | Description |
|---|---|---|---|
| `rtp_route_id` | ID | Yes | Unique route identifier. Prefix `RTPR:` |
| `rtp_agency_id` | ID | Yes | Agency operating this route. Prefix `RTP:` |
| `display_name` | String | Yes | Human-readable route name |
| `service_class` | Enum | Yes | `HOHO` `SCENIC` `CONNECTOR` `REGION` `TRANSFER` |
| `market` | String | Yes | Market code (e.g. `BNE`, `GC`, `SYD`) |
| `published` | Boolean | Yes | Whether this route is publicly visible |
| `gtfs_route_id` | ID | No | Reference to GTFS `route_id` if applicable |

**Service Classes:**

- `HOHO` — Hop-On Hop-Off. Passenger boards/alights freely within validity window.
- `SCENIC` — Fixed itinerary, scheduled departure, scenic focus.
- `CONNECTOR` — Point-to-point transfer between hubs.
- `REGION` — Area access pass. No fixed route; stop cluster defines scope.
- `TRANSFER` — Shuttle or transfer without tourism content.

---

### 2.2 `rtp_stop_context.csv`

Commerce and operational flags per stop. Supplements GTFS `stops.txt`.

| Field | Type | Required | Description |
|---|---|---|---|
| `stop_id` | ID | Yes | Foreign key to GTFS `stop_id` |
| `rtp_route_id` | ID | Yes | Route this context applies to |
| `stop_role` | Enum | Yes | `HUB` `HOTEL` `ATTRACTION` `CLUSTER` `DEPOT` |
| `sales_enabled` | Boolean | Yes | Whether tickets can be sold at this stop |
| `nfc_enabled` | Boolean | Yes | NFC tap-on validation available |
| `qr_enabled` | Boolean | Yes | QR code validation available |
| `pilot_flag` | Boolean | No | Stop is in pilot/trial mode |
| `venue_partner_id` | String | No | Partner venue identifier |

**Stop Roles:**

- `HUB` — Primary boarding/sales point.
- `HOTEL` — Hotel pickup. Passenger-initiated, not operator-staffed.
- `ATTRACTION` — A destination stop with experience content.
- `CLUSTER` — A grouped set of nearby attractions treated as one stop.
- `DEPOT` — Operator vehicle depot. Not passenger-facing.

---

### 2.3 `rtp_products.csv`

Sellable products. The atomic unit of commerce in GTFS-RTP.

| Field | Type | Required | Description |
|---|---|---|---|
| `rtp_product_id` | ID | Yes | Unique product identifier. Prefix `RTPP:` |
| `product_type` | Enum | Yes | `DAY_PASS` `SINGLE_RIDE` `REGION_PASS` `MULTI_DAY` `SEASON` |
| `display_name` | String | Yes | Human-readable product name |
| `currency` | String | Yes | ISO 4217 currency code (e.g. `AUD`) |
| `price_cents` | Integer | Yes | Price in smallest currency unit |
| `validity_rule` | Enum | Yes | `CALENDAR_DAY` `FIXED_WINDOW` `TRIP_BOUND` `OPEN` |
| `validity_value` | String | No | Hours for `FIXED_WINDOW`; empty for others |
| `rtp_route_id` | ID | No | Route this product is scoped to (if applicable) |
| `published` | Boolean | Yes | Whether this product is publicly available |

**Validity Rules:**

- `CALENDAR_DAY` — Valid for the calendar day of purchase/activation.
- `FIXED_WINDOW` — Valid for N hours from activation (value = hours).
- `TRIP_BOUND` — Valid for one specific trip instance only.
- `OPEN` — No expiry. Gift voucher or lifetime access.

---

### 2.4 `rtp_packages.csv`

Bundled products and experience clusters sold as a unit.

| Field | Type | Required | Description |
|---|---|---|---|
| `rtp_package_id` | ID | Yes | Unique package identifier. Prefix `RTPK:` |
| `display_name` | String | Yes | Human-readable package name |
| `package_class` | Enum | Yes | `TRANSPORT_ONLY` `TRANSPORT_PLUS` `EXPERIENCE_ONLY` |
| `includes_transport` | Boolean | Yes | Whether transport is included |
| `included_items` | JSON | Yes | Array of `{type, id}` objects (see below) |
| `rtp_product_ids` | JSON | Yes | Array of `rtp_product_id` strings included |
| `channels_allowed` | JSON | Yes | Array of allowed sales channels |
| `published` | Boolean | Yes | Whether this package is publicly available |

**`included_items` types:**

- `pickup` — A specific pickup stop (`stop_id`)
- `cluster` — An attraction cluster (`stop_id` of type `CLUSTER`)
- `product` — An inline product reference (`rtp_product_id`)
- `experience` — An external experience or operator product (`partner_experience_id`)

**`channels_allowed` values:** `WEB` `APP` `NFC` `AGENT` `PARTNER` `WHOLESALE`

---

### 2.5 `rtp_entitlements.csv`

Issued access tokens. Records what a customer is entitled to and under what scope.

| Field | Type | Required | Description |
|---|---|---|---|
| `rtp_entitlement_id` | ID | Yes | Unique entitlement identifier. Prefix `RTPE:` |
| `entitlement_type` | Enum | Yes | `PRODUCT` `PACKAGE` `COMP` `STAFF` |
| `rtp_product_id` | ID | No | Source product (if product-based) |
| `rtp_package_id` | ID | No | Source package (if package-based) |
| `valid_from` | Unix Timestamp | Yes | Start of validity window |
| `valid_until` | Unix Timestamp | Yes | End of validity window |
| `scope` | Enum | Yes | `REGION` `ROUTE` `TRIP` `STOP` |
| `rtp_route_id` | ID | No | Route scope (if `ROUTE`) |
| `rtp_trip_id` | ID | No | Trip scope (if `TRIP`) |
| `stop_ids` | JSON | No | Stop scope array (if `REGION` or `STOP`) |
| `status` | Enum | Yes | `ACTIVE` `USED` `EXPIRED` `CANCELLED` |

---

### 2.6 `rtp_trip_instances.csv`

Scheduled departure instances for tourism routes.

| Field | Type | Required | Description |
|---|---|---|---|
| `rtp_trip_id` | ID | Yes | Unique trip instance identifier. Prefix `RTPT:` |
| `rtp_route_id` | ID | Yes | Parent route |
| `departure_time` | ISO 8601 | Yes | Scheduled departure datetime (local) |
| `origin_stop_id` | ID | Yes | Origin stop |
| `capacity_total` | Integer | Yes | Total passenger capacity |
| `capacity_available` | Integer | Yes | Current available capacity |
| `status` | Enum | Yes | `SCHEDULED` `BOARDING` `DEPARTED` `COMPLETED` `CANCELLED` |
| `gtfs_trip_id` | ID | No | Reference to GTFS `trip_id` if applicable |

---

### 2.7 `rtp_capacity_snapshot.csv`

Point-in-time capacity state per trip. Used for real-time availability.

| Field | Type | Required | Description |
|---|---|---|---|
| `rtp_trip_id` | ID | Yes | Trip instance |
| `snapshot_time` | Unix Timestamp | Yes | When this snapshot was recorded |
| `capacity_total` | Integer | Yes | Total capacity |
| `capacity_booked` | Integer | Yes | Confirmed bookings |
| `capacity_available` | Integer | Yes | Remaining available |
| `source` | String | Yes | System that generated the snapshot |

---

### 2.8 `rtp_attribution_points.csv`

Commission and partner attribution per transaction.

| Field | Type | Required | Description |
|---|---|---|---|
| `attribution_id` | ID | Yes | Unique attribution record |
| `rtp_entitlement_id` | ID | Yes | Source entitlement |
| `partner_id` | String | Yes | Attributing partner |
| `channel` | String | Yes | Sales channel used |
| `commission_rate` | Decimal | No | Commission percentage (0.0–1.0) |
| `commission_cents` | Integer | No | Commission amount in smallest currency unit |
| `currency` | String | No | ISO 4217 currency code |
| `recorded_at` | Unix Timestamp | Yes | When the sale was recorded |

---

### 2.9 `rtp_event_types.csv`

Standardised telemetry event definitions.

| Field | Type | Required | Description |
|---|---|---|---|
| `event_type_id` | ID | Yes | Event identifier |
| `display_name` | String | Yes | Human-readable event name |
| `category` | Enum | Yes | `COMMERCE` `ACCESS` `TRANSPORT` `EXPERIENCE` `SYSTEM` |
| `description` | String | No | What this event represents |

**Standard event types:**

| `event_type_id` | Category | Description |
|---|---|---|
| `purchase_completed` | COMMERCE | Customer completed a purchase |
| `entitlement_created` | ACCESS | Entitlement issued |
| `entitlement_validated` | ACCESS | Entitlement scanned/checked at stop |
| `entitlement_expired` | ACCESS | Entitlement passed validity window |
| `route_started` | TRANSPORT | Trip departed origin |
| `route_completed` | TRANSPORT | Trip arrived at terminus |
| `passenger_boarded` | TRANSPORT | Passenger boarded at stop |
| `passenger_alighted` | TRANSPORT | Passenger alighted at stop |
| `attraction_entered` | EXPERIENCE | Passenger entered attraction |
| `content_played` | EXPERIENCE | Audio/video content triggered at stop |
| `support_opened` | SYSTEM | Customer support interaction initiated |

---

### 2.10 `rtp_vehicles.csv`

Vehicle registry.

| Field | Type | Required | Description |
|---|---|---|---|
| `rtp_vehicle_id` | ID | Yes | Unique vehicle identifier. Prefix `RTPV:` |
| `display_name` | String | Yes | Vehicle name or number |
| `vehicle_type` | Enum | Yes | `BUS` `MINIBUS` `RAIL` `FERRY` `TRAM` `OTHER` |
| `capacity` | Integer | Yes | Passenger capacity |
| `wheelchair_accessible` | Boolean | Yes | Whether wheelchair accessible |
| `operator_id` | String | No | Operating company reference |
| `registration` | String | No | Vehicle registration number |
| `active` | Boolean | Yes | Whether vehicle is in service |

---

### 2.11 `rtp_trip_vehicle_assignment.csv`

Links vehicle to trip instance.

| Field | Type | Required | Description |
|---|---|---|---|
| `rtp_trip_id` | ID | Yes | Trip instance |
| `rtp_vehicle_id` | ID | Yes | Assigned vehicle |
| `assigned_at` | Unix Timestamp | Yes | When assignment was made |
| `assignment_type` | Enum | Yes | `PLANNED` `REALTIME` `SUBSTITUTE` |

---

## 3. Relationship Diagram

```
GTFS stops.txt ←── rtp_stop_context ──→ rtp_routes
                                              │
                                    ┌─────────┴─────────┐
                               rtp_products        rtp_trip_instances
                                    │                    │
                               rtp_packages    rtp_trip_vehicle_assignment
                                    │                    │
                               rtp_entitlements    rtp_vehicles
                                    │
                          rtp_attribution_points
```

---

## 4. Namespace & ID Conventions

- All RTP identifiers are globally scoped using operator-prefix namespacing: `RTPR:operator_route_name`
- Operators should prefix with their agency code to avoid collision: `RTPR:hopio:bne_tm_hoho`
- A future registry for agency codes is planned.

---

## 5. Relationship to Existing Standards

| Standard | Relationship |
|---|---|
| GTFS Static | GTFS-RTP extends. `rtp_routes` may reference `gtfs_route_id`. `rtp_stop_context` references `stop_id`. |
| GTFS Realtime | `rtp_capacity_snapshot` and `rtp_trip_instances.status` complement GTFS-RT vehicle positions. |
| NeTEx | GTFS-RTP targets simpler, CSV-native tooling for operators who cannot implement NeTEx. |
| SIRI | GTFS-RTP telemetry events may be published via SIRI-ET in future versions. |
| Open API (Ticketing) | `rtp_entitlements` is designed to be serveable via REST API. Reference API spec planned for v0.2. |

---

## 6. Versioning

This document is **v0.1.0-draft**.

GTFS-RTP follows semantic versioning:
- **Patch** — Clarifications, typo fixes, no schema changes.
- **Minor** — New optional fields or tables. Backwards compatible.
- **Major** — Breaking field changes or required field additions.

---

## 7. Open Questions (RFC)

The following are open for community input:

1. Should `rtp_agency_id` be registered centrally or remain self-declared?
2. Should `rtp_packages.included_items` be a separate join table rather than inline JSON?
3. How should multi-currency pricing be handled in `rtp_products`?
4. Should entitlement validation events be part of this spec or delegated to the ticketing API layer?
5. Is `rtp_capacity_snapshot` better suited to a GTFS-RT feed rather than a static CSV?

Open an issue or start a Discussion to contribute.

---

## 8. License

Creative Commons Attribution 4.0 International (CC BY 4.0)  
You are free to use, share, and adapt this specification with attribution.

**Author:** Peter Myers — https://github.com/pmyers-abundance
