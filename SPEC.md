# Mobility Demand Exchange Specification

**Working abbreviation:** MDES
**Version:** 0.4 Draft
**Status:** Draft, revision 4 (see Changelog)

## 0. Changelog

**0.4** (this revision):
- §12 split into 12.1/12.2: the mode tree (§12.1, unchanged) remains the
  required simple case, and a new §12.2 **multi-dimensional mode
  classification** lets a publisher additionally decompose an observation
  into `vehicle`, `propulsion`, `role`, `access`, `operating_form`, and
  `analytical_assignment` — separate, orthogonal attributes instead of one
  tree position. This addresses the taxi/e-scooter/carsharing problem the
  tree can't resolve on its own: the same physical trip is legitimately
  "public transport" in one national survey and "individual transport" in
  another, and a single-tree leaf can't hold both readings at once, only a
  decomposed classification can carry the ambiguity explicitly rather than
  forcing a premature choice.
- §14 methodology: added `source_category`, to record a record's original
  category label in the source dataset's own vocabulary (e.g. a traffic
  count's raw `pedestrian` class), distinct from and traceable against the
  record's own `mode`/`mode_classification` value.
- §25 appendix: added the open item that no reconciliation or validation
  rule yet exists between §12.1 tree leaves and §12.2's
  `analytical_assignment` vocabulary — carried forward rather than resolved
  in this pass.

**0.3** (previous revision):
- §12 mode tree: added `car_share`, `ride_share`, `bike_share` under
  `shared_mobility`, and `heavy_goods_vehicle`/`light_commercial` under
  `freight` — both branches were previously undifferentiated leaves, which
  didn't survive contact with real data (BASt's truck/van classes, a
  ride-share listing, a free-floating carsharing log all needed a specific
  leaf to map onto).
- §12 added `unclassified` as a `mode_class: aggregate` value, distinct
  from `all`: `all` means "aggregated across every mode this source
  distinguishes," `unclassified` means "observed, but which mode wasn't
  resolved" (e.g. BASt's `Son`/other vehicle-class column) — conflating
  the two would silently misrepresent an unknown as a deliberate rollup.
- §12 given the same publisher-extension clause §7.1 already gives units,
  closing that gap from the §25 open-items list.
- §14 `collection_method` vocabulary: added `scraped_service_log`, for
  sources built by scraping a public listing service (no other existing
  value fit — `data_fusion`/`administrative_record` both implied a
  different provenance).

**0.2**:
- Renamed the routed-flow geometry model from "path" to **route** (§10, §5,
  §17) to remove its collision with `path` as a spatial-object *type* (§9).
  `path` now unambiguously means a physical alignment (a footpath, cycleway,
  shared-use trail) that can be observed at/against, distinct from the
  *route* a flow of trips follows.
- Added a sixth demand type, **`modal_share`** (§2.6, §6), for the common
  case where only an aggregated mode-share statistic is published — no
  underlying per-mode flow/count records exist or are being released
  alongside it (e.g. a national MiD mode-share table, a MATSim
  `modestats.csv` summary).
- Added a real **`denominator`** object to the measure model (§7.3),
  replacing the earlier unenforceable prose requirement that shares "must
  state" their denominator, included modes, etc. Required whenever
  `unit = percentage`/`rate` or `statistic = share`, for any demand type.
- §13 (Modal split) rewritten to show both the disaggregated pattern
  (multiple mode-tagged flow records) and the standalone `modal_share`
  pattern as equally valid, not the latter as a special case of the former.
- Known items raised in review but **not** addressed in this pass are
  tracked in the new Appendix (§25), not silently dropped.

## 1. Purpose

The Mobility Demand Exchange Specification defines a common, mode-neutral way to exchange observed, estimated, modelled, and synthetic mobility demand data.

The specification supports demand represented at different levels of detail:

* counts at locations;
* trips originating from a place;
* trips arriving at a place;
* movements between origins and destinations;
* trips or aggregate movements with route geometries;
* demand disaggregated by transport mode and other dimensions;
* standalone aggregate mode-share statistics, published without the underlying disaggregated flows they may have been computed from.

MDES is intended to make mobility-demand datasets easier to publish, combine, compare, validate, and convert between transport-analysis systems.

## 2. Scope

Version 0.2 covers six demand-observation types:

### 2.1 Count

A quantity observed or estimated at a point, link, screenline, stop, station, or other spatial object.

Examples include:

* vehicles crossing a road section;
* cyclists passing a counter;
* passengers boarding at a stop;
* pedestrians entering a station;
* vehicle occupancy.

### 2.2 Origin total

A quantity of trips or movements beginning at a specified spatial object.

Example:

> 12,400 person-trips originated in zone A during an average weekday.

### 2.3 Destination total

A quantity of trips or movements ending at a specified spatial object.

Example:

> 9,800 person-trips arrived in zone B between 07:00 and 10:00.

### 2.4 Origin–destination flow

A quantity moving between an origin and a destination.

Example:

> 2,300 bicycle trips travelled from zone A to zone B.

### 2.5 Routed flow

One or more trips between an origin and destination associated with a route (a geometry, a network-link sequence, or a timestamped trajectory).

Example:

> 450 car trips travelled from zone A to zone B using the specified road-link sequence.

### 2.6 Modal share

An aggregated share of demand attributed to a transport mode (or other dimension value), within a stated reference population, area, and period, published on its own — without requiring the underlying per-mode flow, count, or total records it may have been computed from.

Example:

> Of trips made by residents of Landkreis Grafschaft Bentheim on an average weekday in 2026, 42% were by car.

This differs from disaggregating a `count`/`origin_total`/`destination_total`/`od_flow` by mode (§13): it does not require, and does not imply the existence of, matching per-mode absolute records. A dataset consisting of nothing but `modal_share` records — e.g. five rows, one per mode, for one area and period — is a complete, valid MDES dataset in its own right.

## 3. Design principles

### 3.1 Mode-neutral

The core model must support walking, cycling, private vehicles, public transport, shared mobility, freight, aviation, maritime transport, and modes not yet defined.

### 3.2 Aggregation-neutral

The same conceptual model should support:

* one observed trip;
* an expanded survey trip representing many trips;
* an aggregate OD flow;
* a modelled demand estimate;
* a count at a sensor;
* a standalone aggregate share with no underlying disaggregated records.

### 3.3 Source-transparent

Every observation must state whether it is:

* observed;
* reported;
* estimated;
* imputed;
* modelled;
* simulated;
* synthetic.

### 3.4 Privacy-aware

The specification must support aggregation, suppression, spatial generalization, temporal generalization, and other privacy protections.

It must not require personally identifiable information.

### 3.5 Compatible rather than replacement-oriented

MDES should map to existing standards and formats where appropriate, including:

* OMX;
* GTFS and GTFS-ride;
* DATEX II;
* MDS;
* OGC Moving Features;
* GMNS;
* MATSim;
* SUMO;
* GeoJSON.

MDES should not replace domain-specific standards that already serve their purpose well.

### 3.6 Extensible

Publishers must be able to add domain-specific fields without changing the meaning of the core model.

## 4. Core conceptual model

Every demand record is a `DemandObservation`.

```text
DemandObservation
├── identity
├── demand type
├── measure
├── time
├── spatial references
├── dimensions
├── methodology
├── quality
├── provenance
└── privacy
```

## 5. Minimum required fields

Every demand observation must contain:

| Field         | Description                                      |
| ------------- | ------------------------------------------------ |
| `id`          | Unique identifier within the dataset             |
| `demand_type` | Type of demand observation                       |
| `value`       | Numeric quantity                                 |
| `unit`        | Meaning of the quantity                          |
| `time`        | Time instant, interval, or representative period |
| `data_status` | Observed, estimated, modelled, or similar        |
| `source`      | Dataset or process that produced the record      |

Depending on `demand_type`, the record must also contain:

| Demand type         | Required spatial information        |
| ------------------- | ----------------------------------- |
| `count`             | `location`                          |
| `origin_total`      | `origin`                            |
| `destination_total` | `destination`                       |
| `od_flow`           | `origin` and `destination`          |
| `routed_flow`       | `origin`, `destination`, and `route` |
| `modal_share`       | `spatial.scope` (optional — if absent, the share applies to the dataset's overall `geographic_coverage`, per `feed_info`/provenance metadata) |

Additionally, regardless of `demand_type`: whenever `unit` is `percentage` or
`rate`, or `statistic` is `share`, the record must also contain a
`denominator` object (§7.3).

## 6. Initial demand-type vocabulary

```text
count
origin_total
destination_total
od_flow
routed_flow
modal_share
```

Future versions may introduce specialized types, but they must remain convertible to one of these core forms.

## 7. Measure model

A measure contains:

```json
{
  "value": 2300,
  "unit": "person_trips",
  "statistic": "estimated_total"
}
```

### 7.1 Initial unit vocabulary

```text
persons
person_trips
vehicle_trips
vehicles
boardings
alightings
passenger_kilometres
vehicle_kilometres
tonnes
tonne_kilometres
percentage
rate
```

Publishers may define additional units using namespaced identifiers.

### 7.2 Statistical meaning

The `statistic` field distinguishes values such as:

```text
observed_count
estimated_total
modelled_total
sample_count
mean
median
rate
share
index
```

### 7.3 Denominator model

Whenever `unit` is `percentage` or `rate`, or `statistic` is `share`, the
measure must include a `denominator` object:

```json
{
  "value": 0.42,
  "unit": "percentage",
  "statistic": "share",
  "denominator": {
    "population": "residents",
    "included_modes": ["car", "public_transport", "cycling", "walking", "other"],
    "unknown_mode_treatment": "excluded",
    "time_period": "average_weekday_2026",
    "geographic_scope": "zone:GrafschaftBentheim"
  }
}
```

| Field | Description |
|---|---|
| `population` | Who or what is being counted, e.g. `residents`, `all_travelers`, `commuters` |
| `included_modes` | The complete set of modes the denominator sums over |
| `unknown_mode_treatment` | How trips of unrecorded/unclassified mode were handled: `excluded` \| `included_as_other` \| `redistributed` |
| `time_period` | The period the share is computed over |
| `geographic_scope` | The area the share applies to |

This applies equally to a mode share computed from disaggregated records and
to a standalone `modal_share` observation (§2.6, §13) — in the latter case,
`denominator` is the *only* place that context is recorded, since no sibling
records exist to infer it from.

## 8. Time model

A record may describe:

* a single timestamp;
* a closed or half-open time interval;
* a recurring period;
* a representative period;
* a model year.

Example:

```json
{
  "start": "2026-05-04T07:00:00+02:00",
  "end": "2026-05-04T08:00:00+02:00",
  "time_zone": "Europe/Berlin",
  "representation": "observed_interval"
}
```

Representative periods may include:

```text
average_weekday
average_weekend_day
typical_monday
school_day
annual_average_daily
model_year
```

The exact dates used to calculate a representative period should be recorded in dataset metadata where available.

## 9. Spatial model

A spatial reference may identify:

```text
point
stop
station
node
link
screenline
zone
administrative_area
grid_cell
path
```

`path` denotes a physical alignment — a defined pedestrian path, cycleway,
or shared-use trail segment — usable as a stable location for a `count` or
other observation. It is distinct from the *route* a flow of trips follows
(§10), which is a derived trajectory computed or observed for a specific
demand record, not a standing spatial object.

A `modal_share` record (§2.6) may reference a spatial object via
`spatial.scope` to mean "the reference area this share applies to" — this is
neither an origin nor a destination, just the area/population the share is
computed over.

Spatial objects should have stable identifiers and may additionally include geometries.

Example:

```json
{
  "id": "zone:A",
  "type": "zone",
  "geometry": {
    "type": "Polygon",
    "coordinates": []
  }
}
```

External references may use identifiers from systems such as:

* GTFS;
* GMNS;
* OpenStreetMap;
* national transport networks;
* census geographies;
* administrative boundary systems.

The identifier system and version must be stated.

## 10. Route model

A routed flow may use one or more route representations.

### 10.1 Geometry

```json
{
  "representation": "geometry",
  "geometry": {
    "type": "LineString",
    "coordinates": []
  }
}
```

### 10.2 Network-link sequence

```json
{
  "representation": "network_sequence",
  "network_id": "example-network",
  "network_version": "2026-01",
  "link_ids": ["101", "102", "205", "301"]
}
```

### 10.3 Timestamped trajectory

```json
{
  "representation": "trajectory",
  "positions": [
    {
      "time": "2026-05-04T07:12:00+02:00",
      "longitude": 13.4049,
      "latitude": 52.5200
    }
  ]
}
```

Every route should state its derivation:

```text
observed
reported
map_matched
inferred
modelled
shortest_path
representative
synthetic
```

## 11. Dimensions

Demand observations may be disaggregated using dimensions.

Initial dimensions include:

```text
mode
trip_purpose
vehicle_type
population_segment
user_type
direction
occupancy
service
operator
fare_product
commodity
```

Dimensions must not change the fundamental meaning of the measure.

`mode` takes a value from §12.1's tree. A record may additionally, or
instead, carry `mode_classification` (§12.2), a decomposed object rather
than a single tree value.

## 12. Mode model

### 12.1 Simple mode

Mode is represented as a hierarchical classification.

Example:

```text
all
├── active
│   ├── walking
│   ├── cycling
│   └── micromobility
├── private_motorized
│   ├── car_driver
│   ├── car_passenger
│   └── motorcycle
├── public_transport
│   ├── bus
│   ├── tram
│   ├── metro
│   ├── rail
│   └── ferry
├── shared_mobility
│   ├── car_share
│   ├── ride_share
│   └── bike_share
├── freight
│   ├── light_commercial
│   └── heavy_goods_vehicle
└── unclassified
```

`unclassified` (`mode_class: aggregate`) means an observation counted a
trip/vehicle but could not resolve which mode it was — distinct from
`all`, which means a deliberate aggregate across every mode a source
*does* distinguish. Using `all` for a genuinely unresolved observation
would misrepresent it as an intentional rollup.

A record may contain:

* a primary mode;
* a main mode;
* multiple modal stages;
* an unspecified or aggregated mode.

The specification must distinguish a complete-trip mode from the mode of one stage within a multimodal trip.

Publishers may add mode leaves under any existing branch using namespaced
identifiers, the same way §7.1 allows publisher-defined units — the tree
above is a starting vocabulary, not a closed one.

### 12.2 Multi-dimensional mode classification

A single tree position stops being mutually exclusive once electrification,
sharing, and platform-mediated services are taken into account. The same
taxi trip is counted as public transport in one national travel survey and
as individual transport in another; the same trip on a borrowed e-bike
differs from an owned one only in `access`, yet a tree forces both onto one
leaf or none. Comparing datasets that made different choices under §12.1
means guessing which of several conflated attributes drove the choice.

A publisher who needs their classification to stay comparable across
surveys, tools, or jurisdictions may provide `dimensions.mode_classification`
— an object that separates the attributes a single tree leaf conflates,
using the observed person's role as the closest analogue to a "main mode":

| Field | Answers | Initial vocabulary |
| --- | --- | --- |
| `vehicle` | What physically moves the person, if anything? | `none`, `bicycle`, `car`, `bus`, `rail`, `ship`, `aircraft`, `skateboard`, ... |
| `propulsion` | What produces the movement? | `human_power`, `electric_assist`, `battery_electric`, `combustion`, `hybrid` |
| `role` | What role does the observed person have? | `pedestrian`, `rider`, `driver`, `passenger`, `public_transport_passenger` |
| `access` | How is the vehicle accessed? | `owned`, `household`, `shared`, `rented` |
| `operating_form` | How is the trip organised? | `individual`, `scheduled_service`, `taxi`, `ride_hailing`, `demand_responsive` |
| `analytical_assignment` | Which headline modal-split group does this observation count toward? | `walking`, `cycling`, `miv`, `public_transport`, `other` |

As with §7.1 and §12.1, each vocabulary is a starting point, extensible with
namespaced publisher-defined values.

`analytical_assignment` is intentionally a separate, smaller vocabulary from
§12.1's tree, not an alias for its top-level branch names: it exists to
answer one question — which of the handful of groups a modal-split headline
figure usually reports does this observation belong to — while the tree
serves finer-grained bookkeeping. A publisher using both must state how
their §12.1 leaves map onto §12.2's `analytical_assignment` values (e.g.
whether `shared_mobility/car_share` counts as `miv` or `other`); MDES does
not mandate one mapping, since national practice on this point genuinely
differs (this is the same taxi/carsharing ambiguity `analytical_assignment`
exists to make explicit instead of hiding).

Example — a person riding a borrowed, electrically assisted skateboard,
registered as a pedestrian by a passing traffic counter:

```json
{
  "dimensions": {
    "mode_classification": {
      "role": "rider",
      "vehicle": "skateboard",
      "propulsion": "electric_assist",
      "access": "household",
      "operating_form": "individual",
      "analytical_assignment": "walking"
    }
  },
  "methodology": {
    "source_category": "pedestrian"
  }
}
```

`dimensions.mode` (§11, §12.1) and `dimensions.mode_classification` may be
used independently or together. When both are present on the same record,
`mode_classification.analytical_assignment` should be consistent with
`mode`'s top-level branch, allowing for `analytical_assignment` splitting
`active` into `walking`/`cycling` where the tree does not.

## 13. Modal split

Modal split can be represented two ways, both first-class:

**Disaggregated** — demand observations of another type (`od_flow`,
`origin_total`, `destination_total`, `count`), each carrying a `mode`
dimension, from which shares can be computed by the consumer:

```json
[
  {
    "demand_type": "od_flow",
    "origin": "zone:A",
    "destination": "zone:B",
    "mode": "walking",
    "value": 400,
    "unit": "person_trips"
  },
  {
    "demand_type": "od_flow",
    "origin": "zone:A",
    "destination": "zone:B",
    "mode": "cycling",
    "value": 250,
    "unit": "person_trips"
  }
]
```

**Standalone** (`modal_share`, §2.6) — the share itself is the only thing
published; no per-mode absolute records exist or need to exist:

```json
{
  "demand_type": "modal_share",
  "dimensions": { "mode": "car" },
  "spatial": { "scope": { "id": "zone:GrafschaftBentheim", "type": "administrative_area" } },
  "measure": {
    "value": 0.42,
    "unit": "percentage",
    "statistic": "share",
    "denominator": {
      "population": "residents",
      "included_modes": ["car", "public_transport", "cycling", "walking", "other"],
      "unknown_mode_treatment": "excluded",
      "time_period": "average_weekday_2026",
      "geographic_scope": "zone:GrafschaftBentheim"
    }
  }
}
```

A consumer must not assume the disaggregated pattern is always reconstructable from a `modal_share` dataset, and must not assume a `modal_share` dataset is incomplete for lacking the absolute counts behind it — this is the common shape of most publicly released mode-share statistics (household travel surveys, simulation summary outputs).

## 14. Methodology and quality

Each observation should support:

```text
collection_method
data_status
sample_size
expansion_factor
confidence_interval
standard_error
coverage
missing_data_status
quality_flag
validation_method
source_category
```

`source_category` records the record's original category label in the
source dataset's own vocabulary (e.g. a traffic count's raw `pedestrian`
class) — kept distinct from, and traceable against, the record's own
`mode`/`mode_classification` (§12) so a reclassification into MDES's
vocabulary never silently discards what the source actually said.

Initial `collection_method` values may include:

```text
manual_count
automatic_counter
ticketing_system
vehicle_detection
household_travel_survey
intercept_survey
mobile_device_data
gps_trace
operator_record
administrative_record
transport_model
simulation
data_fusion
scraped_service_log
```

## 15. Provenance

Dataset and record provenance should include:

```text
publisher
creator
source_dataset
source_record
publication_date
dataset_version
processing_method
processing_software
license
attribution
contact
```

Derived records should identify their source records or source datasets where feasible.

## 16. Privacy

The specification should support:

```text
privacy_level
aggregation_method
suppression_status
suppression_reason
spatial_resolution
temporal_resolution
minimum_group_size
noise_method
privacy_notes
```

Suggested privacy levels:

```text
open_aggregate
restricted_aggregate
anonymized_record
synthetic_record
confidential
```

Raw device identifiers, names, addresses, account identifiers, and other direct personal identifiers are outside the core specification.

## 17. Example OD-flow record

```json
{
  "schema_version": "0.2",
  "id": "obs-12345",
  "demand_type": "od_flow",

  "measure": {
    "value": 2300,
    "unit": "person_trips",
    "statistic": "estimated_total"
  },

  "time": {
    "start": "2026-05-04T07:00:00+02:00",
    "end": "2026-05-04T08:00:00+02:00",
    "time_zone": "Europe/Berlin",
    "representation": "average_weekday"
  },

  "spatial": {
    "origin": {
      "id": "zone:A",
      "type": "zone"
    },
    "destination": {
      "id": "zone:B",
      "type": "zone"
    }
  },

  "dimensions": {
    "mode": "public_transport",
    "trip_purpose": "all"
  },

  "methodology": {
    "data_status": "estimated",
    "collection_method": "household_travel_survey",
    "sample_size": 412,
    "expansion_factor_applied": true
  },

  "provenance": {
    "publisher": "Example Transport Authority",
    "source_dataset": "Regional Travel Survey 2026",
    "license": "CC-BY-4.0"
  },

  "privacy": {
    "privacy_level": "open_aggregate"
  }
}
```

## 18. Example modal-share record

```json
{
  "schema_version": "0.2",
  "id": "obs-67890",
  "demand_type": "modal_share",

  "measure": {
    "value": 0.42,
    "unit": "percentage",
    "statistic": "share",
    "denominator": {
      "population": "residents",
      "included_modes": ["car", "public_transport", "cycling", "walking", "other"],
      "unknown_mode_treatment": "excluded",
      "time_period": "average_weekday_2026",
      "geographic_scope": "zone:GrafschaftBentheim"
    }
  },

  "time": {
    "start": "2026-01-01",
    "end": "2026-12-31",
    "representation": "annual_average_daily"
  },

  "spatial": {
    "scope": {
      "id": "zone:GrafschaftBentheim",
      "type": "administrative_area"
    }
  },

  "dimensions": {
    "mode": "car"
  },

  "methodology": {
    "data_status": "modelled",
    "collection_method": "simulation"
  },

  "provenance": {
    "publisher": "Modalyzer",
    "source_dataset": "Grafschaft Bentheim 2026 MATSim run — modestats.csv",
    "license": "CC-BY-4.0"
  },

  "privacy": {
    "privacy_level": "open_aggregate"
  }
}
```

No sibling `od_flow`/`count` records back this number — the MATSim run's
`modestats.csv` only ever reports the converged district-wide mode shares
themselves, which is exactly the case §13's standalone pattern exists for.

## 19. Version 0.2 implementation profiles

Version 0.2 should define three implementable profiles.

### Profile A: Tabular aggregate demand

Designed for CSV, Parquet, and database tables.

Supports:

* counts;
* origin totals;
* destination totals;
* OD flows;
* modal shares (disaggregated-by-mode or standalone).

### Profile B: JSON demand records

Designed for APIs, document databases, and datasets with nested metadata.

Supports all core demand types.

### Profile C: Routed demand

Designed for trip records and aggregate route flows.

Supports:

* GeoJSON route geometries;
* network-link sequences;
* timestamped trajectories;
* route derivation metadata.

## 20. Version 0.2 deliverables

The initial release should contain:

1. A normative conceptual specification.
2. A JSON Schema.
3. A CSV profile and data dictionary.
4. A controlled vocabulary for modes, units, methods, and statuses.
5. Six example datasets.
6. Validation rules.
7. Mapping guides for selected existing standards.
8. A permissively licensed reference validator.
9. A public issue and governance process.

## 21. Initial example datasets

The test suite should include:

1. Bicycle-counter observations.
2. Public-transport boarding and alighting counts.
3. A zone-to-zone commuting matrix.
4. A household travel-survey trip table.
5. Modelled routed demand on a road network.
6. A published aggregate modal-share table with no underlying disaggregated flows (e.g. a national or district-wide mode-share summary).

A successful core model must represent all six without changing its fundamental structure.

## 22. Initial validation rules

A record is invalid when:

* its `demand_type` is unknown;
* a required spatial reference is absent;
* its unit is missing;
* its time meaning is unspecified;
* its value is not numeric;
* a percentage/rate value or a `share` statistic lacks a `denominator` object;
* a route references an unidentified network;
* an estimated value is presented as observed;
* incompatible dimensions are combined;
* provenance is absent.

Additional warnings should be produced for:

* negative demand values;
* percentages outside the expected range;
* overlapping categories;
* inconsistent modal totals;
* incomplete temporal coverage;
* unknown coordinate-reference systems;
* unresolved spatial identifiers.

## 23. Governance principles

The project should be:

* openly governed;
* publicly documented;
* implementation-driven;
* versioned;
* backward-conscious;
* independent of a single software vendor;
* licensed for unrestricted public implementation.

Changes should be proposed through public issues and reviewed against real datasets.

## 24. Definition of success

Version 0.4 succeeds when an implementer can use one model to publish and validate:

* a traffic count;
* a trip-origin total;
* a trip-destination total;
* an OD matrix;
* a mode-specific OD matrix;
* individual trip records;
* routed aggregate demand;
* a standalone aggregate mode-share statistic;

while preserving the meaning, provenance, uncertainty, temporal basis, and privacy status of each dataset.

## 25. Appendix: open items from prior review, not addressed in this revision

Raised during review but out of scope for this pass — tracked here so they
aren't silently lost:

- **Tabular flattening rules for Profile A.** Nested objects
  (`spatial`, `time`, `dimensions`, `methodology`, `provenance`, `privacy`)
  have no defined column-naming convention for CSV/Parquet; two
  implementations of Profile A could produce incompatible layouts.
- **Multimodal trip legs.** §12 requires distinguishing a complete-trip mode
  from a single stage's mode, but no leg/stage record type exists yet.
- **Time-representation ambiguity.** §8/§17 don't state whether a dated
  interval tagged with a representative-period label (e.g.
  `average_weekday`) is one real sampled day or an arbitrary placeholder.
- **Profile boundary for `routed_flow`.** Unclear whether a tabular
  (Profile A) file may ever carry a `routed_flow` record (e.g. a link-id
  list in one column), or whether that's strictly Profile C's territory.
- **Missing validation rule.** §22 doesn't flag a `mode` value outside the
  declared taxonomy, unlike its equivalent checks for `demand_type` and
  spatial/network identifiers.
- **`schema_version` placement.** Present in the worked examples (§17, §18)
  but not listed in §5's normative minimum-fields table.
- **No §12.1/§12.2 reconciliation rule.** §12.2 requires a stated mapping
  from §12.1 tree leaves to `analytical_assignment` values but doesn't
  define its shape, and §22 has no validation rule catching a record where
  `mode` and `mode_classification.analytical_assignment` disagree.

Resolved in v0.3: mode taxonomy extensibility (§12.1 now has the same
namespaced-extension clause as §7.1's units).

Resolved in v0.4: multi-dimensional mode classification (§12.2), addressing
the taxi/carsharing/e-scooter cross-survey ambiguity previously left to
§12.1's single tree.
