Yes. I would define **one Earth Flow reference architecture with four deployment profiles** rather than four independent architectures.

The invariant is the Earth Flow contract:

```text
Sources
   ↓
Ingestion
   ↓
Raw Earth Data
   ↓
Lakehouse / Earth Catalog
   ↓
Geospatial + Raster Processing
   ↓
Features / Earth State
   ↓
Events + Models
   ↓
Earth Products
   ↓
API / MCP
   ↓
Agents + Applications
```

The implementation changes by platform.

---

# 1. Standards we should keep everywhere

These are more important than any particular vendor.

| Concern | Earth Flow standard |
|---|---|
| Tabular schema | Pydantic + JSON Schema |
| Analytical tables | Iceberg or Delta depending platform |
| Vector geo | GeoParquet + WKB/WKT |
| Raster | COG + Zarr |
| Earth imagery metadata | **STAC** |
| Spatial indexing | **H3** |
| Time | UTC + explicit observation/validity timestamps |
| Events | EarthEvent schema, JSON/Avro |
| API | REST/OpenAPI |
| Agent tools | **MCP** |
| Ontology | RDF/OWL-compatible canonical definitions |
| ML lifecycle | MLflow |
| App/backend | Python + FastAPI |
| Frontend | Next.js |
| Maps | MapLibre + deck.gl |
| Infrastructure | Terraform |
| CI/CD | GitHub Actions |

The most important ones for Earth Flow are probably:

```text
STAC
  → describes Earth-observation assets

H3
  → common spatial indexing

COG / Zarr
  → raster representation

GeoParquet
  → portable vector data

Iceberg / Delta
  → analytical data products

MCP + REST
  → application/agent boundary
```

That means an `irrigation_recommendation` product should have essentially the **same logical schema** whether its physical implementation is Iceberg on AWS, Delta on Databricks, or Delta in OneLake.

---

# 2. The high-level mapping

| Capability | AWS | Databricks | Fabric | Local / OSS |
|---|---|---|---|---|
| Object storage | S3 | Cloud object storage | OneLake | RustFS / Ozone |
| Lakehouse | Iceberg / S3 Tables | Delta Lake | Delta Lake | Apache Iceberg |
| Catalog | Glue Catalog | Unity Catalog | OneLake Catalog | Apache Polaris |
| Governance | Lake Formation | Unity Catalog | OneLake Catalog + Purview | OpenMetadata |
| Batch ingestion | Glue / Python | Lakeflow Connect | Data Factory | Python + Dagster |
| Stream ingestion | Kinesis / MSK | Lakeflow / Spark Streaming | Eventstream | Kafka |
| Stream compute | Managed Flink | Spark Structured Streaming | Eventstream / Eventhouse | Flink |
| Batch compute | Glue / EMR | Spark + Photon | Fabric Spark | Spark / DuckDB |
| Geospatial | Sedona + Python | Native H3 + Spark | Spark + Python libs | Sedona + DuckDB Spatial |
| Raster | Rasterio/Xarray | Rasterio/Xarray | Rasterio/Xarray | Rasterio/Xarray |
| SQL analytics | Athena | Databricks SQL | SQL endpoint / Warehouse | Trino |
| Real-time analytics | Flink/Athena | Streaming tables | Eventhouse/KQL | Flink/Trino |
| Operational DB | Aurora/PostGIS | Lakebase | Fabric SQL DB | PostgreSQL/PostGIS |
| Graph | Neptune | GraphFrames / external graph | Fabric Graph | Neo4j / Oxigraph |
| Ontology | RDF + Neptune | UC semantics + RDF | **Fabric IQ Ontology** | RDF/OWL + Oxigraph |
| ML | SageMaker AI | Mosaic AI / MLflow | Fabric Data Science | MLflow |
| Vector/RAG | OpenSearch | AI Search | Fabric IQ / Foundry | pgvector/Qdrant |
| Agents | Bedrock + AgentCore | Agent Bricks | Fabric Data Agent + Foundry | LangGraph/PydanticAI |
| BI | QuickSight | AI/BI | Power BI | Superset/Metabase |
| Observability | CloudWatch | System tables/MLflow | Monitoring Hub | Prometheus/Grafana |

Now the interesting part is what I would actually choose within each stack.

---

# 3. AWS version

AWS should be the most **open-format-first cloud implementation**.

## Architecture

```text
                        EARTH PRODUCTS

             Farm / Water / Forest / Climate
                         │
                    FastAPI API
                         │
                 MCP / Agent Tools
                         │
            Bedrock + AgentCore
                         │
      ┌──────────────────┼──────────────────┐
      │                  │                  │
   Athena             Neptune          OpenSearch
      │
      └──────────────────┬──────────────────┘
                         │
                  Earth Products
                         │
                Features / Events
                         │
               Glue / EMR / Flink
                         │
                ┌────────┴────────┐
                │                 │
           S3 Tables             S3
          Apache Iceberg     COG / Zarr
                │
         Glue Data Catalog
                │
         Lake Formation
```

Amazon S3 Tables is now a strong foundation because it stores tables natively as Apache Iceberg and integrates with engines such as Athena and Spark. AWS also exposes S3 Tables through Iceberg-compatible interfaces. [AWS Documentation](https://docs.aws.amazon.com/AmazonS3/latest/userguide/s3-tables.html?utm_source=chatgpt.com)

## AWS tool selection

### Storage

**Amazon S3**

Use normal S3 buckets for:

```text
raw/
rasters/
documents/
models/
exports/
```

Use **S3 Tables** for:

```text
core.*
features.*
products.*
events.*
```

Example:

```text
S3
├── earthflow-raw/
│   ├── sentinel/
│   ├── landsat/
│   ├── weather/
│   └── knowledge/
│
├── earthflow-raster/
│   ├── cog/
│   └── zarr/
│
└── S3 Tables
    ├── core
    ├── features
    └── products
```

S3 Tables supports Iceberg, including newer Iceberg v3 capabilities. I would still keep **H3 + portable geometries** in the canonical Earth Flow schema rather than tying the model to a vendor's geometry implementation. [AWS Documentation](https://docs.aws.amazon.com/AmazonS3/latest/userguide/s3-tables.html?utm_source=chatgpt.com)

---

### Catalog + governance

Use:

```text
AWS Glue Data Catalog
+
Lake Formation
```

Glue becomes the technical catalog.

Lake Formation handles:

```text
column permissions
row permissions
table permissions
domain access
cross-account access
```

Lake Formation provides fine-grained governance across Athena, EMR, Glue and related AWS analytics services. [AWS Documentation](https://docs.aws.amazon.com/lake-formation/latest/dg/service-integrations.html?utm_source=chatgpt.com)

---

### Batch ingestion

Use primarily:

```text
AWS Glue
```

For API-based sources:

```text
EventBridge
    ↓
Lambda / ECS task
    ↓
S3 raw
    ↓
Glue transformation
```

Examples:

```text
INMET API
NASA API
STAC catalogs
MapBiomas downloads
IBGE
ANA
```

---

### Large-scale processing

Start with:

```text
Glue Spark
```

Move heavy Earth workloads to:

```text
EMR Serverless
```

particularly for:

```text
large Sentinel archives
large H3 transformations
spatial joins
regional/global processing
```

EMR Serverless provides managed Spark without maintaining a persistent cluster. [AWS Documentation](https://docs.aws.amazon.com/emr/?utm_source=chatgpt.com)

---

### Geospatial

Use:

```text
Apache Sedona
H3
GeoPandas
Shapely
PyProj
```

and:

```text
Rasterio
Xarray
rioxarray
stackstac
```

for raster workloads.

Example:

```text
Sentinel STAC
     ↓
stackstac
     ↓
Xarray
     ↓
NDVI
     ↓
H3 aggregation
     ↓
Iceberg

features.vegetation_health
```

---

### Streaming

Use either:

```text
Kinesis
```

for AWS-first architecture,

or:

```text
MSK / Kafka
```

if Kafka portability matters.

Processing:

```text
Managed Service for Apache Flink
```

Example:

```text
Weather station
IoT sensor
Satellite event
     ↓
Kinesis
     ↓
Flink
     ↓
EarthEvent
     ↓
Iceberg
```

---

### SQL

Use:

```text
Athena
```

for analytical querying.

Example:

```sql
SELECT
    h3_cell,
    avg(soil_moisture),
    avg(ndvi)
FROM features.earth_cell_daily
WHERE observed_date >= current_date - interval '30' day
GROUP BY h3_cell;
```

---

### Operational application database

Use:

```text
Aurora PostgreSQL
+
PostGIS
```

for:

```text
users
farms
field polygons
subscriptions
agent state
application configuration
alerts
```

Don't make the lakehouse your OLTP database.

---

### Earth Graph

Use:

```text
Amazon Neptune
```

Neptune supports property graphs through openCypher/Gremlin and RDF graphs through SPARQL, which makes it particularly suitable if Earth Flow keeps its ontology in RDF/OWL-compatible form. [AWS Documentation](https://docs.aws.amazon.com/neptune/latest/userguide/intro.html?trk=article-ssr-frontend-pulse_little-text-block\&utm_source=chatgpt.com)

Example:

```text
Farm
  grows → Coffee

Coffee
  susceptibleTo → HeatStress

HeatWave
  affects → Municipality

Municipality
  contains → Farm
```

---

### AI / agents

I'd use:

```text
Amazon Bedrock
+
Bedrock AgentCore
```

with Earth Flow exposed as MCP tools.

AgentCore currently supports remote MCP servers as a tool mechanism, which fits the portable Earth Flow tool strategy. [AWS Documentation](https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/harness-tools.html?sc_channel=el\&trk=1ad04439-1c50-4fdd-a845-d07d2655fe7a\&utm_source=chatgpt.com)

For RAG:

```text
OpenSearch
```

for vector + semantic retrieval. OpenSearch supports vector search and RAG-oriented workloads. [AWS Documentation](https://docs.aws.amazon.com/opensearch-service/latest/developerguide/vector-search.html?utm_source=chatgpt.com)

---

# 4. Databricks version

This is probably the **cleanest single-platform implementation**.

```text
                        EARTH PRODUCTS
                              │
                       Databricks Apps
                              │
                        Agent Bricks
                              │
                ┌─────────────┼───────────────┐
                │             │               │
             AI Search     Lakebase     Databricks SQL
                │                             │
                └─────────────┬───────────────┘
                              │
                       Gold Products
                              │
                      Feature Tables
                              │
              Lakeflow Declarative Pipelines
                              │
                        Spark / Photon
                              │
                          Delta Lake
                              │
                       Unity Catalog
```

Lakeflow is now Databricks' unified data-engineering family covering ingestion, pipelines and orchestration, while Unity Catalog provides governance, permissions, lineage and cataloging. [Databricks Documentation](https://docs.databricks.com/aws/en/data-engineering/?utm_source=chatgpt.com)

---

## Storage

Default:

```text
Delta Lake
```

I would **not force Iceberg into the Databricks implementation** simply because AWS uses it.

Use:

```text
Bronze
Silver
Gold
```

mapped to Earth Flow:

```text
bronze   → raw
silver   → core
gold     → features/products
```

If interoperability becomes important, introduce Iceberg/compatibility at the boundaries.

---

## Catalog + governance

Use:

```text
Unity Catalog
```

for:

```text
tables
volumes
models
functions
agents
lineage
permissions
```

That means the structure could become:

```text
earthflow
│
├── raw
├── core
├── features
├── products
├── events
└── knowledge
```

---

## Ingestion

Use:

```text
Lakeflow Connect
```

for supported sources.

Use:

```text
Auto Loader
```

for files arriving into storage.

For Earth APIs and STAC:

```text
Python task
    ↓
Lakeflow
    ↓
Delta
```

---

# 5. Databricks geospatial layer

Databricks is particularly attractive now because H3 is available directly in the platform.

```text
lat/lon
   ↓
H3
   ↓
earth_cell
```

Current Databricks runtimes expose native H3 functions, including indexing, parent/children relationships, rings, distance and polygon coverage. [Databricks Documentation](https://docs.databricks.com/aws/pt/sql/language-manual/sql-ref-h3-geospatial-functions?utm_source=chatgpt.com)

Example:

```sql
SELECT
    h3_longlatash3(longitude, latitude, 9)
FROM observations;
```

Databricks also now has native `GEOGRAPHY` support, although some capabilities remain preview, so **H3 should remain the stable cross-platform spatial key**. [Databricks Documentation](https://docs.databricks.com/aws/en/sql/language-manual/data-types/geography-type?utm_source=chatgpt.com)

For complex processing:

```text
H3
+
Sedona if required
+
GeoPandas
+
Rasterio/Xarray
```

---

# 6. Databricks streaming

Use:

```text
Spark Structured Streaming
+
Lakeflow pipelines
```

Example:

```text
Event stream
    ↓
Streaming Table
    ↓
feature calculation
    ↓
EarthEvent
```

Databricks Lakeflow pipelines can contain both streaming tables and materialized views under Unity Catalog governance. [Databricks Documentation](https://docs.databricks.com/gcp/en/ldp/unity-catalog?utm_source=chatgpt.com)

---

# 7. Databricks serving

For analytical serving:

```text
Databricks SQL
```

For operational application state:

```text
Lakebase
```

Lakebase now provides managed Postgres and can sync/serve lakehouse data to low-latency applications. [Databricks Documentation](https://docs.databricks.com/aws/en/oltp/projects/lakehouse-integrations?utm_source=chatgpt.com)

I'd therefore use:

```text
Delta
→ analytical truth

Lakebase
→ app-facing state
```

---

# 8. Databricks AI stack

This is where this deployment becomes particularly compelling.

```text
Agent Bricks
+
AI Search
+
MLflow
+
Unity Gateway
+
Lakebase
```

Databricks Agent Bricks currently provides managed agent deployment, persistent state/memory, tool integration, MCP support and MLflow-based tracing/evaluation. [Databricks Documentation](https://docs.databricks.com/aws/en/agents?utm_source=chatgpt.com)

For RAG:

```text
AI Search
```

It can create search indexes directly from governed lakehouse tables. [Databricks Documentation](https://docs.databricks.com/aws/en/ai-search/ai-search/?utm_source=chatgpt.com)

Agent:

```text
Earth Agent
    │
    ├── MCP → weather
    ├── MCP → field state
    ├── MCP → water balance
    ├── MCP → satellite
    └── AI Search → knowledge
```

This would probably be the **most elegant Earth Flow implementation for a portfolio demonstration of Lakehouse + Streaming + AI Agents**.

---

# 9. Microsoft Fabric version

Fabric requires a slightly different design philosophy.

Its strongest differentiator isn't Spark.

It's the combination:

```text
OneLake
+
Real-Time Intelligence
+
Power BI
+
Fabric IQ
+
Ontology / Graph
+
Agents
```

That is extremely interesting for Earth Flow.

```text
                           Applications
                                │
                 Power BI / Custom App / Agents
                                │
                           Fabric IQ
                    ┌───────────┼──────────┐
                    │           │          │
                Ontology      Graph    Data Agent
                    │
       ┌────────────┴──────────────────────┐
       │                                   │
   Lakehouse                           Eventhouse
       │                                   │
   Historical                         Real-time
       │                                   │
       └─────────────────┬─────────────────┘
                         │
                      OneLake
```

---

# 10. Fabric storage

Use:

```text
OneLake
+
Fabric Lakehouse
+
Delta Lake
```

for historical analytical state.

Fabric explicitly positions Lakehouse for big-data/ML workloads, Eventhouse for high-throughput streaming data, SQL Database for transactional workloads, and OneLake as the common foundation. [Microsoft Learn](https://learn.microsoft.com/en-us/fabric/fundamentals/store-data?utm_source=chatgpt.com)

Structure:

```text
EarthFlow Lakehouse

Files/
├── raster/
├── cog/
├── zarr/
└── knowledge/

Tables/
├── core_weather
├── core_field
├── feature_vegetation
├── feature_water_balance
└── product_field_state
```

---

# 11. Fabric ingestion

Batch:

```text
Data Factory Pipelines
```

For low/no-code ingestion:

```text
Dataflow Gen2
```

For zero-copy access:

```text
OneLake Shortcuts
```

For complex Earth data:

```text
Fabric Notebook
+
Python/Spark
```

Microsoft's current guidance explicitly separates these patterns based on ingestion type. [Microsoft Learn](https://learn.microsoft.com/en-us/fabric/data-engineering/load-data-lakehouse?utm_source=chatgpt.com)

---

# 12. Fabric streaming

This is one of Fabric's best areas.

Use:

```text
Eventstream
    ↓
Eventhouse
    ↓
KQL
```

for:

```text
weather stations
IoT
alerts
fires
sensor streams
EarthEvent streams
```

Eventstreams can route incoming events into an Eventhouse or Lakehouse and integrate with Fabric's real-time tooling. [Microsoft Learn](https://learn.microsoft.com/pt-br/fabric/real-time-intelligence/event-streams/overview?utm_source=chatgpt.com)

Earth Flow:

```text
Sensor
   ↓
Eventstream
   ↓
Eventhouse
   ↓
EarthEvent
   ↓
Operations Agent
   ↓
Alert
```

---

# 13. Fabric geospatial

Here I would stay deliberately open.

Use Fabric Spark notebooks with:

```text
H3 Python
GeoPandas
Shapely
PyProj
Rasterio
Xarray
Sedona where appropriate
```

Write derived results back into the Lakehouse.

I would **not make Earth Flow dependent on a Fabric-specific geospatial representation**.

H3 stays:

```text
EarthCell = H3
```

everywhere.

---

# 14. Fabric ontology is extremely interesting

Earth Flow's:

```text
Earth Graph
+
Ontology
```

maps almost perfectly onto **Fabric IQ**.

Fabric IQ currently provides ontology, graph, data-agent and operations-agent concepts over OneLake data. Ontology is designed to define entities, properties, relationships, rules and data bindings. [Microsoft Learn](https://learn.microsoft.com/en-us/fabric/iq/?utm_source=chatgpt.com)

We could define:

```text
Field
Farm
Crop
River
Watershed
Forest
Species
ClimateEvent
```

relationships:

```text
Farm CONTAINS Field
Field GROWS Crop
Field LOCATED_IN Watershed
Drought AFFECTS Field
River FEEDS Reservoir
Forest HABITAT_FOR Species
```

And then use a Fabric Data Agent over it.

Fabric Data Agents can currently query lakehouses, warehouses, semantic models, KQL databases and ontologies. [Microsoft Learn](https://learn.microsoft.com/en-us/fabric/data-science/how-to-create-data-agent?utm_source=chatgpt.com)

That makes **Fabric arguably the most interesting ontology/semantic-layer implementation** of the four.

One caveat: parts of Fabric IQ, including ontology, remain preview as of October 2026. I would therefore keep the canonical Earth Flow ontology in version-controlled RDF/OWL/YAML and treat the Fabric ontology as a deployed representation of that model. [Microsoft Learn](https://learn.microsoft.com/en-us/fabric/iq/ontology/overview?utm_source=chatgpt.com)

---

# 15. Fabric application stack

Use:

```text
Power BI
```

for analytics.

Use:

```text
Real-Time Dashboard
```

for event monitoring.

Use:

```text
Fabric Data Agent
```

for conversational structured-data queries.

Use:

```text
Microsoft Foundry
```

when you need a larger custom agent orchestration layer.

---

# 16. Local / OSS version

This is important because I think **Earth Flow should have an implementation that can run without any cloud vendor**.

I'd make this the development reference implementation.

```text
                     Next.js / MapLibre
                            │
                         FastAPI
                            │
                          MCP
                            │
          ┌─────────────────┼─────────────────┐
          │                 │                 │
       Trino           PostgreSQL          Qdrant
          │             + PostGIS             │
          │                                   │
       Iceberg                              Agent
          │
       Polaris
          │
       RustFS
```

Processing:

```text
Dagster
   │
   ├── DuckDB
   ├── Spark
   ├── Sedona
   ├── Flink
   └── Python
```

---

# 17. Local storage

I would use:

```text
RustFS
```

for simple S3-compatible local object storage.

Alternative for more infrastructure-heavy deployments:

```text
Apache Ozone
```

I would **not start a new OSS architecture on MinIO in 2026**. The original MinIO OSS project reached end-of-life and its repository was archived in April 2026. [GitHub](https://github.com/minio/minio?pubDate=20260222\&utm_source=chatgpt.com)

Interestingly, Apache Polaris' current local examples themselves use RustFS, Ozone and Ceph as object-storage options. [Apache Polaris](https://polaris.apache.org/guides/?utm_source=chatgpt.com)

---

# 18. Local lakehouse

Use:

```text
Apache Iceberg
```

for tables.

```text
Apache Polaris
```

for the REST catalog.

Polaris implements the Iceberg REST API and works with Spark, Flink, Trino and other engines. [Apache Polaris](https://polaris.apache.org/?utm_source=chatgpt.com)

Architecture:

```text
RustFS
   │
Iceberg
   │
Polaris
   │
 ┌─┴──────────────┐
 │                │
Trino           Spark
 │                │
SQL             ETL
```

This is a very clean modern lakehouse.

---

# 19. Local processing

For **small data**:

```text
DuckDB
+
DuckDB Spatial
```

For **medium/large geospatial**:

```text
Apache Sedona
```

For distributed transformation:

```text
Spark
```

For raster:

```text
Rasterio
Xarray
rioxarray
stackstac
GDAL
```

I would avoid Spark for everything.

Think:

```text
< ~50 GB
DuckDB / Python

large vector joins
Sedona

large distributed ETL
Spark
```

---

# 20. Local orchestration

Use:

```text
Dagster
```

because Earth Flow is naturally asset-oriented:

```text
raw_weather
    ↓
weather_observation
    ↓
earth_cell_weather
    ↓
water_balance
    ↓
field_state
    ↓
irrigation_recommendation
```

This maps much better to assets than to a giant collection of cron scripts.

---

# 21. Local streaming

Use:

```text
Apache Kafka
+
Apache Flink
```

Architecture:

```text
Weather event
Satellite event
Sensor event
     ↓
Kafka
     ↓
Flink
     ↓
EarthEvent
     ↓
Iceberg
```

Don't introduce Kafka/Flink for the first hackathon build, though.

Add it when we demonstrate the streaming architecture.

---

# 22. Local query engine

Use:

```text
Trino
```

for the shared SQL engine.

Use DuckDB for:

```text
notebooks
local exploration
small pipelines
tests
```

So:

```text
Trino = platform query engine
DuckDB = developer query engine
```

---

# 23. Local operational database

Use:

```text
PostgreSQL
+
PostGIS
```

This should store operational application data:

```text
users
farms
fields
saved areas
alerts
subscriptions
agent sessions
STAC metadata if using pgSTAC
```

Not the massive analytical history.

---

# 24. Local Earth Graph

Two good options.

### Property graph

```text
Neo4j Community
```

### Semantic/RDF graph

```text
Oxigraph
+
RDFLib
```

I prefer the second initially because Earth Flow's canonical semantic model can then use:

```text
RDF
OWL
SHACL
SPARQL
```

and map naturally to Neptune or Fabric IQ later.

---

# 25. Local AI stack

For a completely local setup:

```text
vLLM / Ollama
        │
     model
        │
 LangGraph / PydanticAI
        │
      MCP
        │
  Earth Flow Tools
```

RAG:

```text
Qdrant
```

or simply:

```text
PostgreSQL + pgvector
```

for an initially smaller platform.

Agent observability:

```text
MLflow
+
OpenTelemetry
```

---

# 26. The Earth-specific libraries should be identical everywhere

This deserves its own package.

```text
earthflow-geo/
```

Dependencies:

```text
h3
geopandas
shapely
pyproj

rasterio
rioxarray
xarray
stackstac
pystac
pystac-client

numpy
pandas
polars
```

Potential structure:

```text
earthflow/
├── geo/
│   ├── h3.py
│   ├── raster.py
│   ├── vector.py
│   ├── stac.py
│   └── crs.py
│
├── climate/
├── water/
├── vegetation/
├── agriculture/
└── events/
```

This logic should not live inside Databricks notebooks or Glue scripts.

Those environments should **call the package**.

That preserves portability.

---

# 27. The agent tools should also be identical

This is another critical architectural boundary.

Create:

```text
earthflow-mcp
```

with:

```text
get_weather()

get_weather_forecast()

get_earth_state()

get_field_state()

get_vegetation_health()

get_water_balance()

get_active_events()

calculate_irrigation()

calculate_crop_risk()

get_climate_history()

search_knowledge()
```

Then:

### AWS

```text
Bedrock AgentCore
     ↓ MCP
Earth Flow
```

### Databricks

```text
Agent Bricks
     ↓ MCP
Earth Flow
```

### Fabric

```text
Foundry / Data Agent
     ↓ MCP/API
Earth Flow
```

### Local

```text
LangGraph
     ↓ MCP
Earth Flow
```

The agent isn't the architecture.

**The tools are.**

---

# 28. Same data model everywhere

For example:

```text
products.field_state
```

should remain conceptually:

```text
field_id
observed_date

h3_cell

crop_id
crop_stage

temperature_mean
temperature_max

rainfall_7d
rainfall_30d

soil_moisture

ndvi
ndvi_delta

water_deficit

drought_score
heat_score
disease_score

quality_score

source_version
calculated_at
```

On AWS:

```text
Iceberg
```

Databricks:

```text
Delta
```

Fabric:

```text
Delta
```

OSS:

```text
Iceberg
```

Application code doesn't care.

---

# 29. Repository structure

I would now make Earth Flow look like this:

```text
earth-flow/
│
├── packages/
│   ├── earthflow-core/
│   ├── earthflow-geo/
│   ├── earthflow-climate/
│   ├── earthflow-agriculture/
│   ├── earthflow-water/
│   ├── earthflow-events/
│   ├── earthflow-ontology/
│   └── earthflow-mcp/
│
├── ingestion/
│   ├── sentinel/
│   ├── landsat/
│   ├── inmet/
│   ├── ana/
│   ├── mapbiomas/
│   └── nasa/
│
├── products/
│   ├── earth-state/
│   ├── field-state/
│   ├── water-balance/
│   ├── vegetation-health/
│   ├── climate-risk/
│   └── irrigation/
│
├── apps/
│   ├── web/
│   └── api/
│
├── platforms/
│   ├── aws/
│   ├── databricks/
│   ├── fabric/
│   └── local/
│
└── infrastructure/
    ├── terraform/
    ├── docker/
    └── kubernetes/
```

And:

```text
platforms/aws
```

contains deployment code, **not domain logic**.

---

# 30. My recommended profile for each

### AWS — best for cloud architecture

Use when you want to showcase:

```text
open lakehouse
serverless
cloud-native
event-driven
Iceberg
AWS architecture
```

Core:

```text
S3 + S3 Tables
Glue
Lake Formation
Athena
EMR
Kinesis/Flink
Aurora/PostGIS
Neptune
Bedrock/AgentCore
```

---

### Databricks — best overall Earth Flow platform

Use when you want:

```text
Lakehouse
+
Streaming
+
ML
+
Agents
```

in the most coherent environment.

Core:

```text
Delta Lake
Unity Catalog
Lakeflow
Spark/Photon
H3
Databricks SQL
Lakebase
MLflow
AI Search
Agent Bricks
```

Databricks would currently be **my first choice for a full Earth Flow hosted implementation**.

---

### Fabric — best semantic/operational intelligence version

Its killer architecture is:

```text
OneLake
+
Lakehouse
+
Eventhouse
+
Fabric IQ
+
Ontology
+
Data Agents
+
Power BI
```

I would use Fabric specifically to demonstrate that:

> Earth Flow isn't only a data lake. It has a semantic Earth model that both humans and agents can reason over.

That's quite compelling.

---

### Local / OSS — reference implementation

Core:

```text
RustFS
Iceberg
Polaris
Dagster
DuckDB
Spark
Sedona
Kafka
Flink
Trino
Postgres/PostGIS
Oxigraph
MLflow
FastAPI
MCP
MapLibre
```

This should be the implementation developers can clone and run.

Apache Polaris itself now documents local stacks built around the same Iceberg REST architecture, including RustFS/Ozone + Polaris + Trino/Spark/Flink. [Apache Polaris](https://polaris.apache.org/guides/?utm_source=chatgpt.com)

---

# 31. What I would actually build

I wouldn't build four implementations simultaneously.

I would establish the project in this order:

```text
                 EARTH FLOW CORE

              Python domain packages
                       +
                 Earth schemas
                       +
                  H3 / STAC
                       +
                     MCP
                       │
          ┌────────────┴─────────────┐
          │                          │

      LOCAL / OSS              DATABRICKS
   reference stack            hosted stack

          │                          │
          └────────────┬─────────────┘
                       │

                later adapters

                 AWS       Fabric
```

**Local/OSS first** proves that Earth Flow itself isn't a cloud wrapper.

**Databricks second** gives you the strongest Modern Data Stack / Lakehouse / Streaming / AI portfolio implementation.

Then:

**AWS** demonstrates hyperscaler-native architecture.

**Fabric** demonstrates OneLake + real-time + ontology/semantic intelligence.

The result is much more interesting than four diagrams of equivalent services: **Earth Flow becomes a portable Earth intelligence architecture with multiple execution backends.**

Would you like to define the canonical Earth Flow schemas and interfaces next, or map the hackathon MVP onto one of these stacks?