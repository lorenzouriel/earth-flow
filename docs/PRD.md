# Earth Flow

## Product Requirements Document

**Status:** Draft  
**Version:** 0.1  
**Initial use case:** Reboot the Earth São Paulo 2026  
**Initial domain:** Agriculture  
**Long-term scope:** Environmental and planetary intelligence infrastructure

---

# 1. Executive Summary

Earth Flow is an open data and intelligence platform designed to transform fragmented Earth observation, environmental, agricultural, hydrological, climate, and socioeconomic datasets into reusable products that developers, researchers, governments, communities, and AI systems can consume.

Earth observation data is abundant. Satellite imagery, weather observations, hydrological measurements, soil datasets, climate projections, agricultural statistics, land-use maps, biodiversity observations, and economic datasets are produced by hundreds of organizations around the world.

The problem is not simply access to data.

The larger problem is converting heterogeneous data into information that can answer practical questions such as:

- Should this farm irrigate this week?
- Where is vegetation deteriorating?
- Which communities are most exposed to drought?
- Where is flood risk increasing?
- Which agricultural areas are becoming unsuitable for their current crops?
- Where should restoration efforts be prioritized?
- How much carbon is stored or being lost in a particular region?
- Where are ecosystems recovering?
- Which environmental events require intervention?

Today, teams attempting to answer these questions repeatedly rebuild the same underlying infrastructure:

1. discover datasets;
2. download or connect to them;
3. normalize schemas;
4. resolve geospatial differences;
5. process raster imagery;
6. create time series;
7. calculate indicators;
8. train models;
9. develop APIs;
10. build applications.

Earth Flow moves this reusable work into a shared platform.

The platform continuously ingests Earth data, harmonizes it around common spatial and temporal models, derives environmental features, detects events, runs models, maintains provenance, and exposes the resulting intelligence through APIs and AI-compatible tools.

Applications such as agricultural copilots, drought monitoring systems, restoration planning tools, forest monitoring systems, and carbon intelligence products can then be built on top of the same infrastructure.

The initial Reboot the Earth implementation focuses on agriculture and demonstrates Earth Flow through two applications:

- **Farm Copilot** — identifies climate and crop risks affecting a farm.
- **Water Optimizer** — estimates irrigation needs and potential water savings.

These applications are not standalone systems. They demonstrate how multiple climate products can consume the same underlying Earth Flow data products.

---

# 2. Product Vision

## 2.1 Vision

Create an open intelligence layer for understanding the changing state of the Earth.

Earth Flow should allow developers and AI systems to move from:

> “Where can I find the data required to answer this environmental question?”

to:

> “Ask Earth Flow for the environmental state, risks, changes, or forecasts for this place.”

The platform should progressively become capable of representing:

- what exists in a location;
- what has happened there;
- what is happening now;
- what has changed;
- what risks exist;
- what is likely to happen;
- what interventions may be appropriate.

---

# 3. Product Thesis

Earth Flow is based on five assumptions.

### 3.1 Environmental data infrastructure is repeatedly rebuilt

Agriculture, climate risk, forestry, water management, disaster monitoring, conservation, and carbon projects often use overlapping data sources and calculations.

These projects should share infrastructure.

### 3.2 Earth data is fundamentally spatiotemporal

Most environmental observations can be represented as:

```text
variable
+
location
+
time
+
value
+
source
+
uncertainty
```

Examples include:

```text
temperature
São Paulo
2026-10-27 12:00
31.2°C
INMET
```

or:

```text
NDVI
H3 cell 88a810...
2026-10-25
0.72
Sentinel-2
```

A shared spatiotemporal model therefore allows different environmental domains to interoperate.

### 3.3 Applications should consume products, not raw datasets

Applications should not have to understand:

- Sentinel bands;
- INMET schemas;
- Landsat scenes;
- MapBiomas classification codes;
- coordinate systems;
- raster tiling;
- weather-station normalization.

Earth Flow should expose higher-level products such as:

```text
vegetation_health
water_balance
drought_risk
crop_water_requirement
land_change
fire_risk
```

### 3.4 Scientific computation and generative AI should remain separate

LLMs should explain, orchestrate, retrieve, and interact.

They should not replace deterministic environmental calculations.

For example:

```text
weather
+
crop coefficient
+
evapotranspiration
+
soil moisture
+
rain forecast
        ↓
water balance model
        ↓
recommended irrigation: 14 mm
        ↓
LLM
        ↓
human-readable explanation
```

### 3.5 Environmental intelligence should be reusable

A drought model developed for agriculture may also support:

- water-resource planning;
- ecosystem monitoring;
- fire-risk assessment;
- food-security analysis.

Earth Flow therefore treats derived environmental intelligence as reusable infrastructure.

---

# 4. Product Principles

Earth Flow should follow the following principles.

## 4.1 Open by default

Prefer:

- open standards;
- open table formats;
- open geospatial formats;
- documented APIs;
- reproducible transformations;
- public datasets where possible.

The platform should avoid unnecessary dependency on a single cloud or vendor.

---

## 4.2 Provenance first

Every derived value should be traceable.

For example:

```text
irrigation_recommendation
        ↓
water_balance_v3
        ↓
soil_moisture_estimation_v2
weather_forecast_v4
crop_coefficient_v1
        ↓
specific datasets and timestamps
```

Users should be able to understand where a result came from.

---

## 4.3 Uncertainty should be explicit

Earth Flow must not present uncertain environmental estimates as absolute facts.

Data products should support:

```text
confidence
quality_score
coverage
source
observation_age
model_version
```

---

## 4.4 Products over pipelines

The primary output should not be ETL jobs.

The primary output should be reusable data and intelligence products.

---

## 4.5 One platform, multiple domains

Agriculture is the initial implementation, not the architectural boundary.

---

# 5. Target Users

Earth Flow serves several categories of users.

## 5.1 Developers

Developers building:

- climate applications;
- agricultural tools;
- environmental dashboards;
- research systems;
- AI agents;
- disaster-management applications.

Their primary interface is the API and SDK.

---

## 5.2 Data scientists and researchers

Users exploring:

- climate trends;
- vegetation change;
- agricultural productivity;
- water availability;
- land degradation;
- carbon;
- biodiversity.

Their primary interfaces are SQL, notebooks, lakehouse tables, and APIs.

---

## 5.3 Governments and NGOs

Organizations needing:

- environmental monitoring;
- climate-risk analysis;
- conservation planning;
- agricultural assistance;
- disaster preparedness.

Their interface may be dashboards, reports, APIs, or custom applications.

---

## 5.4 Farmers and agricultural organizations

Users needing practical decisions rather than raw environmental data.

Examples:

- irrigation decisions;
- climate alerts;
- crop risk;
- disease conditions;
- land-health monitoring.

---

## 5.5 AI systems

AI agents should become first-class consumers of Earth Flow.

Instead of allowing AI models unrestricted database access, Earth Flow should expose well-defined environmental tools.

Example:

```text
get_weather()
get_water_state()
get_vegetation_health()
get_climate_risk()
detect_land_change()
calculate_irrigation()
```

---

# 6. Core Platform Architecture

Earth Flow consists of seven major layers.

```text
Earth Products
      ↓
Intelligence
      ↓
Events
      ↓
Earth Graph
      ↓
Features
      ↓
Lakehouse
      ↓
Earth Data
```

---

# 7. Earth Data Layer

The ingestion layer connects Earth Flow to external data sources.

Initial categories include:

## Weather

Examples:

- meteorological stations;
- precipitation;
- temperature;
- humidity;
- wind;
- solar radiation;
- weather forecasts.

Potential sources include national meteorological agencies and global climate datasets.

---

## Satellite

Examples:

- Sentinel;
- Landsat;
- MODIS;
- other Earth-observation systems.

Satellite data supports:

- vegetation monitoring;
- surface water;
- land cover;
- fire;
- crop health;
- soil exposure;
- urban heat.

---

## Hydrology

Examples:

- rivers;
- reservoirs;
- watersheds;
- water levels;
- streamflow;
- groundwater where available.

---

## Soil

Examples:

- soil classification;
- texture;
- organic carbon;
- moisture;
- fertility;
- erosion susceptibility.

---

## Land Use

Examples:

- forests;
- agriculture;
- pasture;
- urban areas;
- wetlands;
- protected areas.

---

## Agriculture

Examples:

- crop locations;
- production;
- planting calendars;
- crop coefficients;
- yield statistics;
- agricultural practices.

---

## Biodiversity

Examples:

- species observations;
- habitats;
- protected areas;
- ecological corridors.

---

## Human Systems

Examples:

- population;
- infrastructure;
- administrative boundaries;
- markets;
- agricultural prices.

These datasets allow Earth Flow to eventually connect environmental conditions with human consequences.

---

# 8. Lakehouse

The lakehouse stores structured environmental data and derived products.

Recommended foundational technologies include:

```text
Object Storage

        +

Apache Iceberg

        +

Parquet / GeoParquet
```

Raster data should remain in formats optimized for imagery rather than being forced into relational tables.

Examples:

```text
COG
Zarr
```

The lakehouse should contain four logical layers.

---

## 8.1 Raw

Raw datasets should remain as close as possible to the source representation.

Examples:

```text
raw.inmet_weather
raw.ana_hydrology
raw.ibge_agriculture
raw.mapbiomas_landcover
raw.sentinel_catalog
```

Purpose:

- reproducibility;
- debugging;
- auditing;
- reprocessing.

---

## 8.2 Core

The Core layer standardizes external datasets into Earth Flow's internal model.

Examples:

```text
core.location
core.region
core.field
core.water_body

core.weather_observation
core.soil_observation
core.vegetation_observation
core.hydrology_observation
core.land_cover
```

All observations should contain common metadata where appropriate:

```text
location
timestamp
variable
value
unit
source
source_version
quality
```

---

## 8.3 Features

Feature tables contain reusable analytical indicators.

Examples:

```text
features.temperature_anomaly
features.water_balance
features.soil_moisture
features.vegetation_health
features.land_change
features.drought
features.fire_risk
features.flood_risk
features.carbon
features.crop_suitability
```

Features may be generated through:

- deterministic equations;
- spatial analysis;
- remote-sensing algorithms;
- statistical models;
- machine-learning models.

---

## 8.4 Products

Products represent domain-ready information.

Examples:

```text
products.field_state
products.irrigation_recommendation
products.climate_risk
products.land_change_event
products.crop_suitability
products.restoration_priority
```

Applications should primarily consume this layer.

---

# 9. Earth Cell

A canonical spatial index is required to make datasets interoperable.

Earth Flow should introduce an abstraction called the **Earth Cell**.

An Earth Cell represents a defined geographic area.

Potential implementations include hierarchical spatial indexes such as H3.

Conceptually:

```text
EarthCell

cell_id
geometry
resolution
parent_cell
```

Observations can then be normalized into:

```text
EarthCellObservation

cell_id
timestamp
variable
value
source
quality
```

A derived daily state might resemble:

```text
earth_cell_daily

cell_id
date

temperature_mean
temperature_max
precipitation
humidity

soil_moisture

ndvi
evi
ndwi

surface_water

land_cover

drought_score
heat_score
fire_score
flood_score
```

The Earth Cell provides a shared spatial key across datasets.

---

# 10. Earth Graph

Not every environmental relationship should be represented as a table.

Earth Flow should maintain a semantic graph describing relationships between entities.

Example:

```text
Watershed
    contains → River

River
    feeds → Reservoir

Reservoir
    supplies → Municipality

Municipality
    contains → Farm

Farm
    contains → Field

Field
    grows → Coffee

Coffee
    sensitiveTo → HeatStress
```

Another example:

```text
Forest
    habitatFor → Species

Forest
    stores → Carbon

Fire
    affects → Forest

Drought
    increasesRiskOf → Fire
```

The graph enables reasoning across domains.

The lakehouse answers:

> What observations exist?

The graph answers:

> What is connected to what?

---

# 11. Earth Events

Earth Flow should detect meaningful environmental changes and represent them using a common event model.

```text
EarthEvent

event_id
event_type

geometry

detected_at
started_at
ended_at

severity
confidence

evidence
affected_entities

model_version
```

Initial event types include:

```text
DROUGHT
HEAT_WAVE
FLOOD
FIRE

VEGETATION_LOSS
VEGETATION_RECOVERY

FOREST_LOSS
LAND_USE_CHANGE

CROP_STRESS
WATER_STRESS

SOIL_DEGRADATION

HIGH_DISEASE_RISK
```

Events allow Earth Flow to become proactive.

Instead of applications continuously asking:

> Has something changed?

Earth Flow can notify applications:

> A relevant event has been detected.

---

# 12. Intelligence Layer

The intelligence layer sits above the data products.

It contains:

- deterministic models;
- statistical models;
- ML models;
- forecasting;
- simulation;
- recommendation engines;
- LLM agents.

The architecture should clearly distinguish these responsibilities.

```text
                    Earth Flow Agent

                           │
            ┌──────────────┼──────────────┐
            ↓              ↓              ↓

      deterministic       ML          knowledge
         models          models          base
            │              │              │
            └──────────────┼──────────────┘
                           ↓

                      explanation
```

LLMs must not invent environmental measurements.

They should operate on outputs produced by trusted tools.

---

# 13. Earth Flow API

The API provides a stable interface independent of underlying datasets.

Initial primitives:

```text
get_weather(location, period)

get_weather_forecast(location, period)

get_soil_state(location)

get_water_state(location)

get_vegetation_state(location)

get_land_cover(location)

get_environmental_events(location)

get_time_series(variable, geometry, period)

detect_change(geometry, period)

get_climate_risk(location)

compare_regions(region_a, region_b)
```

Domain-specific tools may include:

```text
calculate_water_balance(field)

calculate_irrigation_need(field)

calculate_crop_risk(field)

calculate_crop_suitability(location, crop)

calculate_carbon_estimate(geometry)

calculate_restoration_priority(geometry)
```

These APIs should also be exposed to AI systems through an agent-compatible tool layer.

---

# 14. Product 1 — Earth Agriculture

Earth Agriculture turns Earth Flow into agricultural intelligence.

Its core object is the agricultural field.

```text
Farm
    ↓
Field
    ↓
Crop Cycle
```

Each field can have a daily derived state.

```text
field_daily_state

field_id
date

crop
crop_stage

temperature
rainfall
humidity

soil_moisture

ndvi
vegetation_change

evapotranspiration
water_deficit

drought_risk
heat_risk
disease_risk

quality_score
```

Earth Agriculture provides infrastructure for several applications.

---

# 15. Farm Copilot

## Problem

Farmers and agricultural organizations often have access to weather forecasts and agricultural recommendations, but the information is fragmented and difficult to convert into decisions specific to a field.

## Product

Farm Copilot continuously evaluates environmental conditions affecting a farm.

Example question:

> What is the greatest risk to my coffee farm this week?

The system combines:

```text
field
+
crop
+
weather forecast
+
historical weather
+
soil
+
vegetation health
+
agronomic knowledge
```

Output:

```text
Primary risk: water stress

Severity: High
Confidence: 84%

Evidence:
- rainfall deficit over previous 21 days
- elevated temperature
- reduced soil moisture
- declining vegetation index

Recommended actions:
...
```

## Key functionality

- farm registration;
- field polygon creation;
- crop registration;
- environmental monitoring;
- weather risk;
- drought risk;
- crop stress detection;
- environmental alerts;
- natural-language questions;
- historical field timeline.

---

# 16. Water Optimizer

## Problem

Irrigation is frequently performed using fixed schedules or intuition rather than current environmental conditions.

This may waste water and energy.

## Product

Water Optimizer estimates when and how much irrigation is required.

Input:

```text
field
crop
crop stage
soil
recent weather
forecast weather
estimated soil moisture
```

Calculation:

```text
reference evapotranspiration
×
crop coefficient
=
crop water demand

crop water demand
-
effective rainfall
-
available soil water
=
irrigation requirement
```

Example output:

```text
Recommended irrigation:
14 mm

Expected rainfall:
8 mm

Estimated irrigation volume:
1,400,000 liters

Estimated avoided irrigation:
600,000 liters

Confidence:
Medium
```

## Important requirement

Water calculations must come from deterministic models.

The LLM may explain the recommendation but must not calculate water demand independently.

---

# 17. Earth Water

Earth Water provides hydrological intelligence.

It combines:

- precipitation;
- rivers;
- reservoirs;
- watersheds;
- groundwater where available;
- soil moisture;
- evapotranspiration;
- surface water;
- climate forecasts.

Primary questions:

- Where is water becoming scarce?
- Which watersheds are under stress?
- Where is flood risk increasing?
- How are reservoirs changing?
- What regions are experiencing abnormal rainfall deficits?

Potential products:

```text
water_availability
watershed_state
reservoir_state
drought_state
flood_risk
surface_water_change
```

---

# 18. Earth Forest

Earth Forest monitors forest and vegetation dynamics.

Inputs include:

- land cover;
- satellite imagery;
- fire observations;
- climate;
- biomass;
- protected areas.

Primary functions:

- detect vegetation loss;
- detect forest loss;
- identify fire events;
- identify vegetation recovery;
- monitor fragmentation;
- estimate environmental pressure.

Example:

```text
FOREST_LOSS

Area:
12.7 hectares

Detected:
2026-10-23

Vegetation change:
-41%

Confidence:
91%
```

Potential users include conservation groups, researchers, governments, and restoration projects.

---

# 19. Earth Carbon

Earth Carbon estimates carbon-related environmental state.

Potential data includes:

- biomass;
- forest cover;
- soil carbon;
- agricultural practices;
- land-use change;
- fire;
- emissions factors.

Potential products:

```text
carbon_stock
carbon_change
land_use_emissions
agricultural_emissions
sequestration_estimate
```

Earth Carbon should distinguish measured values from estimated values.

Every estimate should include:

```text
methodology
source
model_version
confidence
```

---

# 20. Earth Climate

Earth Climate provides historical and forward-looking climate intelligence.

The product combines:

- historical weather;
- climate normals;
- extreme-event observations;
- climate scenarios;
- vulnerability indicators.

It should answer questions such as:

- Is this region becoming hotter?
- How has rainfall variability changed?
- Which crops may become less suitable?
- Where is drought frequency increasing?
- Which communities face increasing heat risk?

Potential products:

```text
temperature_anomaly
rainfall_anomaly
extreme_heat
climate_trend
future_suitability
climate_exposure
```

---

# 21. Earth Restore

Earth Restore helps prioritize ecosystem restoration.

Inputs may include:

- vegetation condition;
- soil degradation;
- erosion;
- water availability;
- biodiversity;
- ecological connectivity;
- climate;
- land use;
- socioeconomic constraints.

The system computes:

```text
restoration_priority
```

Example:

```text
Region A

Degradation severity:     88
Water importance:         76
Biodiversity importance:  91
Connectivity potential:   84
Restoration feasibility:  71

Priority score:           84
```

The product should explain why an area received its score.

---

# 22. Earth Biodiversity

Earth Biodiversity connects species, habitats, ecosystems, and environmental pressures.

Potential capabilities include:

- species distribution;
- habitat condition;
- habitat loss;
- ecological corridors;
- protected-area monitoring;
- environmental threats.

Example relationship:

```text
Species
    dependsOn → Habitat

Habitat
    locatedIn → Ecosystem

Ecosystem
    affectedBy → Drought

Ecosystem
    affectedBy → LandUseChange
```

This product depends heavily on the Earth Graph.

---

# 23. Cross-Product Composition

The greatest value of Earth Flow appears when products are combined.

Example:

```text
Earth Climate
        ↓
drought forecast
        ↓
Earth Water
        ↓
reduced water availability
        ↓
Earth Agriculture
        ↓
crop water stress
        ↓
Farm Copilot
        ↓
farmer alert
```

Another:

```text
Earth Forest
        ↓
forest loss
        ↓
Earth Carbon
        ↓
carbon release estimate
        ↓
Earth Biodiversity
        ↓
habitat impact
        ↓
Earth Restore
        ↓
restoration priority
```

This composability is a primary architectural requirement.

---

# 24. Hackathon MVP

The Reboot the Earth MVP should prove the architecture without attempting to implement the entire vision.

## Scope

The MVP should support:

```text
1 geographic region
1-3 crop types
weather data
basic satellite-derived vegetation data
field polygons
water-balance calculations
climate-risk calculations
```

---

# 25. MVP Data Products

Implement:

```text
core.field
core.crop
core.weather_observation
core.vegetation_observation

features.field_daily
features.water_balance

products.field_state
products.irrigation_recommendation
products.field_risk
```

---

# 26. MVP Applications

## Application A — Water Optimizer

User selects a field.

Display:

- recent rainfall;
- forecast rainfall;
- crop water demand;
- estimated soil-water condition;
- recommended irrigation;
- estimated water savings.

---

## Application B — Farm Copilot

User asks:

> What should I worry about this week?

Earth Flow evaluates:

```text
heat
water
rainfall
vegetation
```

The application returns:

```text
risk ranking
evidence
recommended actions
confidence
```

---

# 27. MVP Demo

The ideal hackathon demonstration is:

### Step 1

Open a map of São Paulo.

### Step 2

Select a farm or predefined field.

### Step 3

Earth Flow displays:

```text
Crop: Coffee

Vegetation health: 78
Water status: 61
Heat risk: High
Drought risk: Medium
```

### Step 4

Ask:

> Preciso irrigar esta semana?

Earth Flow runs the water model.

### Step 5

Result:

```text
Recommendation:
Delay irrigation for approximately 48 hours.

Forecast rainfall:
12 mm

Estimated crop demand:
19 mm

Recommended additional irrigation:
7 mm

Potential avoided irrigation:
~500,000 liters
```

### Step 6

Ask:

> Qual o maior risco para essa plantação?

Farm Copilot explains the most significant risk using the same platform data.

The judges should clearly see that both applications share the same infrastructure.

---

# 28. User Experience Principles

Applications built on Earth Flow should prioritize decisions over raw data.

Bad:

```text
NDVI: 0.61
ET0: 4.8
VPD: 1.6
```

Better:

```text
Vegetation health is slightly below its normal level.

The field has experienced increasing water stress for approximately nine days.

Rainfall is expected within 48 hours, so irrigation can likely be reduced.
```

Expert users should still be able to inspect the underlying measurements.

---

# 29. Explainability

Every recommendation should support a "Why?" view.

Example:

```text
Why is drought risk high?

Rainfall anomaly           -37%
Soil moisture              low
Temperature anomaly        +2.1°C
Vegetation trend           declining
```

Sources should also be visible.

Users must be able to distinguish:

- observed facts;
- model outputs;
- forecasts;
- AI-generated explanations.

---

# 30. Data Provenance

Every product should retain lineage.

Example:

```text
irrigation_recommendation: 7 mm

Derived from:

water_balance_model v0.3

using:

weather_forecast
source: X

soil_estimate
source: Y

crop_coefficient
source: Z

generated:
2026-11-05T13:31:00Z
```

This is necessary for scientific credibility.

---

# 31. Data Quality

Each observation or product should support a quality indicator.

Example:

```text
quality_score: 0.87
```

Potential quality factors:

- source reliability;
- data age;
- spatial resolution;
- cloud contamination;
- missing observations;
- model uncertainty;
- interpolation distance.

Applications should avoid showing false precision.

---

# 32. Platform APIs

Initial API structure:

```text
/v1/locations
/v1/fields
/v1/weather
/v1/vegetation
/v1/water
/v1/events
/v1/risks
```

Examples:

```text
GET /v1/fields/{field_id}/state

GET /v1/fields/{field_id}/weather

GET /v1/fields/{field_id}/vegetation

GET /v1/fields/{field_id}/risks

GET /v1/fields/{field_id}/water-balance

POST /v1/fields/{field_id}/irrigation
```

Later:

```text
GET /v1/regions/{id}/forest

GET /v1/regions/{id}/carbon

GET /v1/regions/{id}/climate

GET /v1/regions/{id}/biodiversity

POST /v1/regions/{id}/restoration-analysis
```

---

# 33. AI Tool Interface

AI agents should use restricted tools.

Example:

```text
get_field_state(field_id)

get_weather_forecast(field_id)

get_water_balance(field_id)

get_vegetation_health(field_id)

get_active_events(field_id)

calculate_irrigation(field_id)

get_crop_guidance(crop, problem)
```

The agent should never silently fabricate missing observations.

If required information is unavailable, it should report that limitation.

---

# 34. Knowledge Layer

Environmental decision-making requires more than numerical observations.

Earth Flow should eventually maintain a knowledge layer containing:

- agricultural manuals;
- scientific literature;
- government recommendations;
- crop guidance;
- environmental standards;
- conservation guidance.

The AI layer can retrieve relevant passages through RAG.

Architecture:

```text
                     Earth Agent

                   /             \

        structured data         knowledge

               ↓                    ↓

          Earth Flow          Knowledge Base
```

Structured data determines what is happening.

The knowledge base helps determine what the condition means and what interventions are documented.

---

# 35. Recommended Technical Architecture

Initial implementation:

```text
Object Storage
     │
     ▼
Apache Iceberg
     │
     ▼
Catalog
     │
     ├── DuckDB / SedonaDB
     ├── Python
     └── Trino
```

Orchestration:

```text
Dagster
```

Geospatial processing:

```text
GeoParquet
COG
Sedona
DuckDB Spatial
```

Serving:

```text
FastAPI
PostgreSQL/PostGIS where operational queries require it
```

Frontend:

```text
Next.js
MapLibre
deck.gl where required
```

AI:

```text
LLM
+
Earth Flow tools
+
knowledge retrieval
```

---

# 36. Repository Structure

```text
earth-flow/
│
├── platform/
│   ├── catalog/
│   ├── storage/
│   ├── orchestration/
│   ├── api/
│   └── ontology/
│
├── ingestion/
│   ├── weather/
│   ├── satellite/
│   ├── hydrology/
│   ├── soil/
│   ├── agriculture/
│   └── landcover/
│
├── core/
│   ├── geography/
│   ├── observations/
│   ├── fields/
│   └── events/
│
├── features/
│   ├── vegetation/
│   ├── climate/
│   ├── water/
│   ├── soil/
│   └── agriculture/
│
├── products/
│   ├── field_state/
│   ├── irrigation/
│   ├── climate_risk/
│   ├── land_change/
│   ├── carbon/
│   └── restoration/
│
├── intelligence/
│   ├── agents/
│   ├── tools/
│   ├── models/
│   └── knowledge/
│
└── apps/
    ├── farm-copilot/
    └── water-optimizer/
```

---

# 37. Non-Goals

Earth Flow should not initially attempt to:

- replace meteorological agencies;
- produce its own global weather model;
- train foundation models for remote sensing;
- become a GIS editor;
- become a generic data warehouse;
- store every Earth dataset available;
- provide fully autonomous agricultural decision-making;
- claim scientific certainty where evidence is incomplete.

Earth Flow should integrate, transform, model, and expose environmental intelligence.

---

# 38. Security and Privacy

Public Earth observation data can generally be openly processed.

Private agricultural data requires stronger controls.

The platform should support:

```text
public
organization
private
```

Private data may include:

- exact farm boundaries;
- proprietary sensor data;
- production information;
- operational agricultural practices.

Access to this information must be isolated by tenant or organization.

Public environmental observations should remain independently usable.

---

# 39. Scientific Governance

Environmental models must be versioned.

Example:

```text
water_balance_v1
water_balance_v2
```

A product result must record the model used.

Changes to scientific algorithms should not silently rewrite historical results without traceability.

Documentation should include:

- assumptions;
- equations;
- datasets;
- known limitations;
- validation methodology.

---

# 40. Open Infrastructure Strategy

Earth Flow should aim to become useful as a shared environmental infrastructure rather than merely a proprietary application.

Potential open components include:

- data schemas;
- ontology;
- API specification;
- ingestion connectors;
- environmental data products;
- model definitions;
- SDKs.

This supports the concept of Earth Flow as potential Digital Public Infrastructure.

---

# 41. Success Metrics

Platform-level metrics:

### Data

```text
number of integrated data sources
geographic coverage
temporal coverage
data freshness
percentage of products with complete lineage
```

### Platform

```text
API latency
pipeline success rate
data-product freshness
number of reusable products
number of applications using each product
```

### Environmental impact

Depending on application:

```text
liters of irrigation avoided
hectares monitored
drought events identified
restoration hectares prioritized
forest-loss events detected
carbon changes estimated
```

### Adoption

```text
developers
organizations
API requests
external applications
open-source contributors
```

---

# 42. Hackathon Success Criteria

The prototype is successful if judges can understand within approximately one minute that:

1. Earth Flow integrates multiple environmental datasets.
2. It converts those datasets into reusable intelligence.
3. Two separate agricultural applications use the same platform.
4. The outputs lead to specific decisions.
5. Environmental impact can be quantified.
6. The architecture can expand beyond agriculture.

The hackathon does not require global coverage or production-scale infrastructure.

---

# 43. Development Phases

## Phase 0 — Hackathon

Build:

```text
weather ingestion
basic satellite integration
field model
field_daily_state
water balance
field risk
Farm Copilot
Water Optimizer
```

Focus:

> Demonstrate the platform thesis.

---

## Phase 1 — Agriculture Platform

Add:

```text
soil
crop calendars
additional crops
historical analytics
disease risk
yield context
event notifications
```

Release a stable Earth Agriculture API.

---

## Phase 2 — Earth Water + Earth Climate

Add:

```text
watersheds
rivers
reservoirs
climate anomalies
drought
flood risk
climate projections
```

---

## Phase 3 — Forest + Carbon + Restore

Add:

```text
land-change detection
forest monitoring
fire
biomass
carbon
restoration prioritization
```

---

## Phase 4 — Earth Graph

Connect:

```text
locations
ecosystems
species
farms
watersheds
environmental events
human infrastructure
```

Enable cross-domain reasoning.

---

## Phase 5 — Earth Intelligence Platform

Expose Earth Flow as infrastructure for external agents and applications.

Developers should be able to ask:

```text
What is happening here?

What changed?

What risks exist?

What is likely to happen?

What systems are affected?

What interventions are available?
```

without rebuilding environmental data infrastructure.

---

# 44. Long-Term Product Model

Earth Flow should eventually look like:

```text
                     EARTH FLOW

                         API

                          │

        ┌─────────────────┼─────────────────┐

        ↓                 ↓                 ↓

     Products           Agents          Developers

        │

 ┌──────┼──────┬──────┬──────┬──────┐
 ↓      ↓      ↓      ↓      ↓      ↓

Farm  Water  Forest Carbon Climate Restore
```

The products share:

```text
Earth data
Earth cells
Earth graph
feature models
environmental events
provenance
AI tools
```

This shared substrate is the core competitive and public value of Earth Flow.

---

# 45. Positioning

Earth Flow should not primarily describe itself as:

> A lakehouse for Earth data.

That describes an implementation detail.

Nor should it be described only as:

> An agricultural AI.

That unnecessarily constrains the vision.

The preferred positioning is:

> **Earth Flow is an open environmental intelligence platform that transforms fragmented Earth data into reusable products, models, events, and APIs for building climate and sustainability applications.**

A shorter version:

> **Earth Flow turns Earth data into actionable intelligence.**

And the developer-oriented framing:

> **Build Earth applications without rebuilding the Earth data stack.**

---

# 46. Reboot the Earth Narrative

The hackathon story should remain deliberately narrow.

### Problem

Farmers must make climate-sensitive decisions using fragmented data.

### Demonstration

Earth Flow integrates:

```text
weather
satellite
crop
soil
```

and derives:

```text
field state
water balance
climate risk
```

Two applications consume those products:

```text
Water Optimizer
Farm Copilot
```

### Broader implication

Agriculture is the first domain.

The same platform architecture can support:

```text
water
forests
climate
carbon
biodiversity
restoration
```

The final message is:

> **We are not building another climate dashboard.**
>
> **We are building the reusable intelligence layer beneath climate applications.**

---

# 47. Core Product Definition

Earth Flow is successful when creating a new environmental application no longer requires a team to rebuild the foundational data stack.

A developer should eventually be able to write something conceptually similar to:

```python
state = earthflow.get_state(location)

risks = earthflow.get_risks(location)

events = earthflow.get_events(location)

forecast = earthflow.forecast(
    location=location,
    variable="water_stress",
    horizon="14d"
)
```

and focus primarily on the problem they are trying to solve.

That is the fundamental product.

**Earth Flow is infrastructure for turning observations of the planet into decisions about the planet.**