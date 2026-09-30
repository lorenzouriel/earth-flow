# PRD — Earth / Weather Data Platform

## 1. Nome do projeto

**Nome provisório:** `EarthFlow Data Platform`

Outras possibilidades:

- TerraFlow
- EarthStream
- Atmos Data Platform
- PlanetOps
- GeoStream
- TerraPulse

Para este PRD, vou usar **EarthFlow**.

---

# 2. Resumo executivo

O **EarthFlow** será uma plataforma de dados ambientais capaz de coletar, processar, armazenar, padronizar e servir dados relacionados ao planeta em múltiplos domínios.

A primeira versão será focada em:

- clima;
- temperatura;
- precipitação;
- vento;
- pressão atmosférica;
- umidade;
- ondas;
- swell;
- maré;
- temperatura da água;
- qualidade do ar;
- eventos ambientais.

A plataforma deverá combinar:

```text
Public APIs
Historical datasets
Meteorological stations
Ocean buoys
Generated sensor data
Streaming events
```

em uma arquitetura moderna:

```text
Ingestion
   ↓
Data Lake
   ↓
Processing
   ↓
Lakehouse
   ↓
Analytics
   ↓
Serving / BI / APIs
```

O objetivo não é apenas criar uma aplicação meteorológica.

O objetivo é construir uma **Data Platform ambiental escalável**, que posteriormente possa receber novos domínios:

```text
Weather
Ocean
Surf
Air Quality
Wildfires
Earthquakes
Volcanoes
Floods
Tornadoes
Hurricanes
Climate
```

---

# 3. Problema

Dados ambientais existem em várias fontes e formatos diferentes.

Exemplos:

```text
REST APIs
CSV
JSON
NetCDF
GRIB
GeoTIFF
Parquet
Sensor streams
```

Cada fonte possui diferenças em:

- frequência;
- unidade;
- resolução temporal;
- resolução espacial;
- schema;
- timezone;
- qualidade;
- disponibilidade histórica.

Isso torna difícil responder perguntas simples de forma consistente.

Exemplo:

> Como estavam as condições de vento, swell, chuva e temperatura em uma determinada região nas últimas 24 horas?

Ou:

> Quais regiões tiveram maior aumento de temperatura nos últimos 30 dias?

Ou:

> Quais praias estão recebendo swell de sudeste com período maior que 10 segundos?

O EarthFlow cria uma camada comum para isso.

---

# 4. Visão

A visão de longo prazo é:

> Criar uma plataforma de dados da Terra capaz de armazenar e analisar eventos ambientais em escala temporal e geográfica.

Arquiteturalmente:

```text
                       EARTHFLOW

                    Data Sources
                         │
          ┌──────────────┼──────────────┐
          │              │              │
       Weather          Ocean       Environment
          │              │              │
          └──────────────┼──────────────┘
                         ▼
                     Ingestion
                         │
                         ▼
                     Data Lake
                         │
                    Bronze Layer
                         │
                         ▼
                     Processing
                    Spark / Flink
                         │
                         ▼
                     Silver Layer
                         │
                         ▼
                      Iceberg
                         │
                         ▼
                      Gold
                         │
             ┌───────────┼───────────┐
             ▼           ▼           ▼
          Athena        API         BI
```

---

# 5. Objetivos

## Objetivos de produto

Construir uma plataforma capaz de:

1. ingerir dados de múltiplas fontes ambientais;
2. armazenar histórico bruto;
3. padronizar schemas e unidades;
4. processar dados batch e streaming;
5. construir datasets analíticos;
6. consultar dados por localização e período;
7. disponibilizar APIs;
8. produzir dashboards;
9. detectar eventos ambientais;
10. permitir expansão para novos domínios.

---

# 6. Objetivos técnicos

O projeto deverá permitir estudar e implementar:

```text
Batch ingestion
Streaming ingestion
Data Lake
Lakehouse
Apache Iceberg
Spark
Flink
Kafka
Airflow
dbt
Data Quality
Geospatial Data
Time Series
Schema Evolution
Observability
Infrastructure as Code
CI/CD
Cloud Architecture
```

---

# 7. Não objetivos iniciais

O MVP não terá:

- previsão meteorológica própria com ML;
- simulação climática;
- modelos físicos atmosféricos;
- aplicativo mobile completo;
- Kubernetes;
- multi-cloud simultâneo;
- análise global em resolução extrema;
- satélite bruto em petabytes.

Esses itens podem entrar depois.

---

# 8. Usuários

## 8.1 Data Engineer

Deseja:

- integrar novas fontes;
- monitorar pipelines;
- fazer backfills;
- consultar dados;
- investigar falhas;
- validar qualidade.

---

## 8.2 Data Analyst

Deseja:

- consultar clima;
- comparar regiões;
- produzir dashboards;
- analisar séries temporais.

---

## 8.3 Data Scientist

Deseja acessar datasets históricos para:

```text
forecasting
anomaly detection
clustering
climate analysis
surf prediction
```

---

## 8.4 Developer

Deseja consumir APIs como:

```http
GET /weather
GET /ocean
GET /locations
GET /surf
```

---

## 8.5 Usuário final

Posteriormente poderá consultar:

```text
weather
waves
surf conditions
air quality
environmental alerts
```

---

# 9. Domínios

A plataforma será organizada por domínio.

```text
EarthFlow
│
├── Geography
│
├── Weather
│
├── Ocean
│
├── Surf
│
├── Air Quality
│
├── Climate
└── Environmental Events
```

---

# 10. Geography Domain

Esse será o domínio base.

Entidades:

```text
planet
continent
country
state
city
location
station
beach
surf_peak
ocean
sea
```

---

# 11. Hierarquia geográfica

```text
Planet
  │
  ▼
Continent
  │
  ▼
Country
  │
  ▼
State / Province
  │
  ▼
City
  │
  ▼
Location
```

Exemplo:

```text
Earth
└── South America
    └── Brazil
        └── São Paulo
            └── Praia Grande
                └── Canto do Forte
```

---

# 12. Tabela `location`

Campos principais:

```text
location_id

name

location_type

country_code
state_code
city

latitude
longitude

elevation

timezone

created_at
updated_at
```

Tipos:

```text
city
weather_station
buoy
beach
surf_peak
airport
sensor
observation_point
```

---

# 13. Weather Domain

Dados principais:

```text
temperature

apparent_temperature

humidity

precipitation

rain

snowfall

cloud_cover

pressure

visibility

wind_speed

wind_direction

wind_gust

weather_code
```

---

# 14. Fact Weather Observation

Tabela:

```text
fact_weather_observation
```

Campos:

```text
observation_id

location_id
station_id

observation_time

temperature_c

apparent_temperature_c

humidity_pct

precipitation_mm

rain_mm

snowfall_mm

pressure_hpa

cloud_cover_pct

visibility_m

wind_speed_ms

wind_direction_deg

wind_gust_ms

source_id

ingested_at
```

---

# 15. Ocean Domain

Dados:

```text
wave_height

wave_direction

wave_period

swell_height

swell_direction

swell_period

secondary_swell

water_temperature

current_speed

current_direction

sea_level
```

---

# 16. Fact Ocean Observation

```text
fact_ocean_observation
```

Campos:

```text
observation_id

location_id
buoy_id

observation_time

wave_height_m

wave_period_s

wave_direction_deg

swell_height_m

swell_period_s

swell_direction_deg

water_temperature_c

current_speed_ms

current_direction_deg

source_id
```

---

# 17. Surf Domain

Esse domínio poderá alimentar diretamente o futuro **Waves for Me**.

Entidades:

```text
beach
surf_peak
surf_condition
surf_score
```

---

# 18. Surf Peak

```text
surf_peak_id

beach_id

name

latitude
longitude

preferred_swell_direction

preferred_wind_direction

preferred_tide

min_wave_height

max_wave_height

skill_level
```

---

# 19. Surf Conditions

Exemplo:

```text
Praia Grande

Wave Height      1.3m

Swell            SE

Period           11s

Wind             NW 7km/h

Tide             rising

Water            21°C
```

---

# 20. Surf Score

Posteriormente poderíamos calcular:

```text
0 → 10
```

Baseado em:

```text
wave height
wave period
swell direction
wind direction
wind speed
tide
```

Exemplo:

```text
Surf Score

8.4 / 10
```

---

# 21. Air Quality Domain

Dados:

```text
PM2.5
PM10

CO

NO2

SO2

O3

AQI
```

Tabela:

```text
fact_air_quality
```

---

# 22. Climate Domain

Inicialmente será derivado de Weather.

Exemplos:

```text
daily average temperature

monthly precipitation

temperature anomaly

heat days

cold days

rain days

wind averages
```

---

# 23. Environmental Events Domain

Fases futuras poderão incluir:

```text
wildfire
earthquake
volcano
flood
tornado
hurricane
storm
lightning
```

Schema genérico:

```text
environment_event
```

Campos:

```text
event_id

event_type

severity

latitude
longitude

start_time
end_time

source

metadata
```

---

# 24. Fontes

A arquitetura deverá suportar:

```text
REST API

files

object storage

stream

database

sensor
```

Tipos:

```text
public API

government source

oceanographic source

meteorological station

synthetic sensor
```

---

# 25. Source Registry

Criar tabela:

```text
data_source
```

Campos:

```text
source_id

name

provider

domain

source_type

base_url

frequency

enabled

last_success

last_failure
```

---

# 26. Ingestion

Teremos dois mecanismos.

```text
Batch

Streaming
```

---

# 27. Batch ingestion

Responsável por:

```text
historical data

daily datasets

API polling

backfill

bulk download
```

Arquitetura:

```text
Public API
    │
    ▼
Airflow
    │
    ▼
Ingestion Job
    │
    ▼
S3 Bronze
```

---

# 28. Streaming ingestion

Para sensores e dados de alta frequência:

```text
Sensor Generator
       │
       ▼
      Kafka
       │
       ▼
      Flink
       │
       ▼
       S3
```

---

# 29. Synthetic Sensors

Uma parte muito importante do projeto.

Como nem todas as fontes possuem alta frequência, poderemos criar sensores virtuais.

Exemplo:

```json
{
  "sensor_id": "BR-SP-OCEAN-001",
  "timestamp": "2026-09-30T12:32:14Z",
  "latitude": -24.008,
  "longitude": -46.412,
  "wave_height": 1.4,
  "wave_period": 11.2,
  "wind_speed": 5.3,
  "water_temperature": 21.8
}
```

---

# 30. Escala configurável

```yaml
simulation:

  sensors: 1000

  events_per_second: 5000
```

Perfis:

```text
Local
100 events/s

Small
1,000 events/s

Medium
5,000 events/s

Large
25,000 events/s

Stress
100,000 events/s
```

---

# 31. Sensor types

```text
weather_station

ocean_buoy

air_quality_station

rain_sensor

wind_sensor

temperature_sensor
```

---

# 32. Sensor state

Para gerar dados realistas, um sensor não deve simplesmente gerar números aleatórios.

Ele terá estado.

Exemplo:

```text
Current temperature

24.2°C
```

Próxima leitura:

```text
24.3°C
```

e não:

```text
3°C
```

sem motivo.

---

# 33. Simulação baseada em random walk

Conceitualmente:

```text
next_value =
current_value
+
trend
+
seasonality
+
noise
```

Isso produzirá séries temporais mais realistas.

---

# 34. Sazonalidade

Exemplo de temperatura:

```text
Daily cycle
+
Seasonal cycle
+
Noise
+
Weather event
```

Conceito:

```text
temperature(t)

      ╭────╮
   ╭──╯    ╰──╮
───╯           ╰───
00  06  12  18  24
```

---

# 35. Eventos simulados

O sistema poderá introduzir:

```text
cold front

heat wave

heavy rain

storm

strong wind

large swell
```

Esses eventos deverão alterar múltiplas variáveis.

Por exemplo:

```text
Cold Front

temperature ↓

pressure ↓

cloud cover ↑

rain ↑

wind ↑
```

---

# 36. Kafka Topics

Estrutura:

```text
earth.weather.observations

earth.ocean.observations

earth.air.observations

earth.sensor.events

earth.environment.events
```

Posteriormente:

```text
earth.earthquake.events

earth.wildfire.events
```

---

# 37. Partition Keys

Weather:

```text
station_id
```

Ocean:

```text
buoy_id
```

Sensor:

```text
sensor_id
```

Geographical events:

```text
geohash
```

---

# 38. Event Envelope

Formato padrão:

```json
{
  "event_id": "evt_19281",

  "event_type": "weather_observation",

  "event_version": "1.0",

  "event_time": "2026-09-30T12:30:00Z",

  "source": "sensor-simulator",

  "location": {
    "latitude": -23.55,
    "longitude": -46.63
  },

  "payload": {}
}
```

---

# 39. Data Lake

Estrutura:

```text
s3://earthflow/
│
├── bronze/
│
├── silver/
│
├── gold/
│
├── quarantine/
│
├── checkpoints/
└── metadata/
```

---

# 40. Bronze

Bronze guarda dados originais.

```text
bronze/

weather/

ocean/

air_quality/

sensors/

environment/
```

Exemplo:

```text
bronze/weather/

year=2026/
month=09/
day=30/
hour=13/
```

---

# 41. Silver

Silver terá dados:

```text
validated

standardized

normalized

deduplicated

geocoded

unit-converted
```

---

# 42. Padronização de unidades

Internamente:

```text
Temperature
Celsius

Wind
meters/second

Precipitation
millimeters

Pressure
hPa

Wave height
meters

Wave period
seconds

Direction
degrees
```

Fontes originais podem usar outras unidades.

---

# 43. Silver tables

```text
silver_weather_observation

silver_ocean_observation

silver_air_quality

silver_station

silver_location
```

---

# 44. Gold

Datasets analíticos.

```text
gold_weather_hourly

gold_weather_daily

gold_weather_monthly

gold_ocean_hourly

gold_surf_conditions

gold_air_quality_daily

gold_location_summary
```

---

# 45. Exemplo — Weather Daily

```text
date

location_id

min_temperature

max_temperature

avg_temperature

total_precipitation

avg_humidity

max_wind_speed

avg_pressure
```

---

# 46. Lakehouse

Formato principal:

```text
Apache Iceberg
```

Arquitetura:

```text
S3
 │
 ▼
Iceberg
 │
 ▼
Glue Data Catalog
 │
 ├── Spark
 ├── Flink
 ├── Athena
 └── Redshift
```

---

# 47. Spark

Responsável principalmente por:

```text
historical processing

large transformations

aggregations

joins

backfills

compaction

reprocessing
```

---

# 48. Flink

Responsável por:

```text
stream processing

real-time aggregation

windowing

stateful processing

event-time processing

late events
```

---

# 49. Stream analytics

Exemplo:

```text
Kafka
  │
  ▼
Flink
  │
  ├── avg temperature / 5m
  │
  ├── rainfall / 1h
  │
  ├── max wind / 10m
  │
  └── wave height / 5m
```

---

# 50. Windowing

Implementar:

```text
Tumbling Windows

Sliding Windows

Session Windows
```

Principalmente:

```text
1 minute

5 minute

1 hour
```

---

# 51. Event Time

Precisaremos separar:

```text
event_time

ingestion_time

processing_time
```

Porque sensores podem enviar dados atrasados.

---

# 52. Late events

Exemplo:

```text
Observation:

12:01

Arrived:

12:09
```

Configuração:

```yaml
streaming:

  watermark: 10m
```

---

# 53. Data Quality

Regras devem existir por domínio.

Weather:

```text
temperature >= -100

temperature <= 70

humidity >= 0

humidity <= 100

pressure > 0
```

Ocean:

```text
wave_height >= 0

wave_period >= 0

direction >= 0

direction <= 360
```

---

# 54. Quarantine

Dados inválidos:

```text
Pipeline
   │
   ├── Valid
   │      ↓
   │    Silver
   │
   └── Invalid
          ↓
       Quarantine
```

---

# 55. Anomalias propositalmente simuladas

Synthetic sensors poderão gerar:

```text
duplicates

missing values

late events

future timestamps

sensor spikes

invalid coordinates

impossible values

schema changes
```

---

# 56. Schema Evolution

Exemplo:

V1:

```json
{
  "temperature": 24
}
```

V2:

```json
{
  "temperature": 24,
  "humidity": 72
}
```

V3:

```json
{
  "temperature": 24,
  "humidity": 72,
  "sensor_quality": 0.98
}
```

---

# 57. Geospatial

Um dos aspectos centrais do EarthFlow.

Cada observação deverá possuir:

```text
latitude

longitude
```

ou:

```text
location_id
```

Podemos posteriormente trabalhar com:

```text
Geohash

H3

GeoJSON
```

---

# 58. H3

H3 pode ser utilizado para dividir o planeta em células.

Conceito:

```text
Earth

⬡⬡⬡⬡⬡
 ⬡⬡⬡⬡
⬡⬡⬡⬡⬡
```

Isso ajuda em:

```text
aggregation

regional queries

heatmaps

spatial partitioning
```

---

# 59. API

Criar uma API de serving.

Endpoints:

```http
GET /locations

GET /weather/current

GET /weather/history

GET /weather/summary

GET /ocean/current

GET /ocean/history

GET /surf/conditions
```

---

# 60. Exemplo

```http
GET /weather/current?lat=-23.55&lon=-46.63
```

Resposta:

```json
{
  "temperature": 24.3,
  "humidity": 70,
  "wind_speed": 4.5,
  "pressure": 1013
}
```

---

# 61. API histórica

```http
GET /weather/history
    ?location_id=123
    &start=2026-09-01
    &end=2026-09-30
```

---

# 62. Query engine

Principal:

```text
Athena
```

Para consultas analíticas:

```sql
SELECT
    date,
    AVG(temperature_c)

FROM weather

WHERE location_id = 123

GROUP BY date;
```

---

# 63. Warehouse

Opcionalmente:

```text
Redshift
```

Principalmente para:

```text
BI

large dashboards

complex analytical queries
```

---

# 64. dbt

Responsável pelos modelos analíticos.

```text
Silver
  │
  ▼
dbt
  │
  ▼
Gold
```

Estrutura:

```text
models/

staging/

intermediate/

marts/
```

---

# 65. Dashboards

QuickSight inicialmente.

Dashboards:

```text
Global Weather

Regional Weather

Ocean Conditions

Surf Conditions

Air Quality

Sensor Health

Pipeline Health
```

---

# 66. Dashboard Weather

KPIs:

```text
Current Temperature

24h Temperature Range

Rainfall

Humidity

Pressure

Wind Speed

Wind Direction
```

---

# 67. Dashboard Ocean

```text
Wave Height

Swell Height

Swell Period

Swell Direction

Water Temperature

Wind
```

---

# 68. Weather Map

Posteriormente:

```text
temperature heatmap

rainfall map

wind map

pressure map
```

---

# 69. Data observability

Monitorar:

```text
records ingested

records processed

pipeline latency

failed records

duplicates

late events

missing locations

API failures
```

---

# 70. Sensor monitoring

```text
sensor online

sensor offline

last event

event rate

invalid readings

latency
```

---

# 71. Kafka monitoring

```text
messages/sec

consumer lag

partitions

producer latency

consumer throughput
```

---

# 72. Spark monitoring

```text
runtime

shuffle

memory

executor usage

task failures

data skew
```

---

# 73. Lake monitoring

```text
number of files

average file size

partitions

storage size

small files
```

---

# 74. AWS Architecture

```text
                      SOURCES

           APIs       Files      Sensors
            │           │           │
            │           │          MSK
            │           │           │
            └──────┬────┴───────────┘
                   │
                   ▼
                  S3
               BRONZE
                   │
          ┌────────┴─────────┐
          │                  │
        Glue/EMR          Flink
         Spark               │
          │                  │
          └────────┬─────────┘
                   ▼
                  S3
               SILVER
                   │
                Iceberg
                   │
             Glue Catalog
                   │
          ┌────────┼─────────┐
          │        │         │
       Athena   Redshift   API
          │
          ▼
      QuickSight
```

---

# 75. AWS Services

Possíveis serviços:

```text
S3

Glue

Athena

EMR Serverless

Managed Service for Apache Flink

MSK

Lambda

ECS/Fargate

Redshift

QuickSight

CloudWatch

IAM

Secrets Manager
```

---

# 76. Orchestration

Inicialmente:

```text
Airflow
```

ou AWS:

```text
MWAA
```

Fluxo:

```text
Extract
  ↓
Validate
  ↓
Bronze
  ↓
Transform
  ↓
Silver
  ↓
Gold
  ↓
Quality
```

---

# 77. Infra as Code

Tudo deverá estar em:

```text
Terraform
```

Estrutura:

```text
terraform/
│
├── networking/
├── s3/
├── kafka/
├── flink/
├── glue/
├── emr/
├── athena/
├── redshift/
├── iam/
└── monitoring/
```

---

# 78. CI/CD

GitHub Actions:

```text
Pull Request
     │
     ▼
Lint
     │
Tests
     │
Terraform Validate
     │
Terraform Plan
     │
Review
     │
Merge
     │
Terraform Apply
     │
Deploy Jobs
```

---

# 79. Ambiente local

Antes da AWS:

```text
Docker Compose
```

Serviços:

```text
Kafka

Flink

Spark

MinIO

PostgreSQL

Airflow

Grafana
```

MinIO será usado como:

```text
local S3
```

---

# 80. Estrutura do repositório

```text
earthflow/
│
├── ingestion/
│   ├── weather/
│   ├── ocean/
│   └── air_quality/
│
├── simulation/
│   ├── sensors/
│   ├── weather/
│   └── ocean/
│
├── streaming/
│   ├── kafka/
│   └── flink/
│
├── processing/
│   └── spark/
│
├── dbt/
│
├── airflow/
│
├── api/
│
├── data-quality/
│
├── schemas/
│
├── infrastructure/
│   ├── docker/
│   └── terraform/
│
├── monitoring/
│
├── tests/
│
└── docs/
```

---

# 81. Epics

## EPIC-01 — Geography Foundation

Construir:

```text
countries

states

cities

locations
```

---

## EPIC-02 — Weather Sources

Criar conectores para dados meteorológicos.

---

## EPIC-03 — Ocean Sources

Criar ingestão de:

```text
waves

swell

water temperature
```

---

## EPIC-04 — Synthetic Sensor Engine

Criar simulador.

---

## EPIC-05 — Kafka Streaming

Criar:

```text
topics

producers

consumers
```

---

## EPIC-06 — Data Lake

Criar:

```text
Bronze

Silver

Gold
```

---

## EPIC-07 — Spark Processing

Criar transformações batch.

---

## EPIC-08 — Flink Streaming

Criar realtime processing.

---

## EPIC-09 — Iceberg Lakehouse

Criar tabelas transacionais.

---

## EPIC-10 — Data Quality

Criar validações.

---

## EPIC-11 — Geospatial Layer

Criar:

```text
latitude / longitude

geohash

H3
```

---

## EPIC-12 — Analytics

Criar:

```text
hourly

daily

monthly
```

---

## EPIC-13 — Surf Analytics

Criar condições e surf score.

---

## EPIC-14 — API

Criar acesso programático aos dados.

---

## EPIC-15 — BI

Dashboards.

---

## EPIC-16 — Observability

Monitoring.

---

## EPIC-17 — Infrastructure

Terraform.

---

## EPIC-18 — CI/CD

Automação.

---

# 82. User stories principais

### STORY-001

```text
As a data engineer

I want to ingest weather observations

So that they can be stored historically.
```

Acceptance criteria:

```text
data stored in Bronze

source identified

event timestamp preserved

ingestion timestamp recorded

retry supported
```

---

### STORY-002

```text
As a platform

I want to normalize weather units

So that all sources can be compared.
```

Acceptance criteria:

```text
temperature → Celsius

wind → m/s

pressure → hPa

precipitation → mm
```

---

### STORY-003

```text
As a data engineer

I want to generate synthetic sensors

So that streaming pipelines can be tested at scale.
```

Acceptance criteria:

```text
configurable sensors

configurable event rate

deterministic seed

realistic time series

Kafka output
```

---

### STORY-004

```text
As an analyst

I want hourly weather datasets

So that I can analyze regional trends.
```

---

### STORY-005

```text
As a surfer

I want surf conditions by location

So that I can compare surf spots.
```

---

# 83. Data contracts

Cada domínio possuirá contratos versionados.

Exemplo:

```text
schemas/

weather/
  weather-observation-v1.json

ocean/
  ocean-observation-v1.json
```

---

# 84. SLOs iniciais

Batch:

```text
Data freshness:

< 1 hour
```

Streaming:

```text
Processing latency:

< 30 seconds
```

API:

```text
p95 latency:

< 500 ms
```

Esses valores inicialmente são metas técnicas, não requisitos rígidos de produção.

---

# 85. Escala do MVP

Uma boa meta:

```text
Locations

50,000

Weather observations

100M+

Ocean observations

20M+

Synthetic sensor events

100M+

Streaming

1K-5K events/s
```

---

# 86. MVP — versão 1

Eu manteria o MVP relativamente focado.

### Domínios

```text
Geography

Weather

Ocean

Surf
```

### Fontes

```text
2 weather sources

1 ocean source

synthetic sensor stream
```

### Arquitetura

```text
API ingestion
     │
     ▼
Data Lake
     │
     ▼
Spark
     │
     ▼
Iceberg
     │
     ▼
Athena
```

Streaming:

```text
Sensor Simulator
       │
       ▼
      Kafka
       │
       ▼
      Flink
       │
       ▼
    Iceberg
```

---

# 87. Roadmap

## Phase 1 — Geography

Criar:

```text
country

state

city

location

beach

surf_peak
```

---

## Phase 2 — Weather ingestion

```text
Weather API
     │
     ▼
Python ingestion
     │
     ▼
S3 Bronze
```

---

## Phase 3 — Data Lake

Criar:

```text
Bronze

Silver

Gold
```

---

## Phase 4 — Spark

Construir:

```text
weather normalization

hourly aggregates

daily aggregates
```

---

## Phase 5 — Iceberg

Migrar Silver e Gold.

---

## Phase 6 — Ocean

Adicionar:

```text
wave

swell

temperature
```

---

## Phase 7 — Synthetic Sensors

Gerar:

```text
weather sensor events

ocean buoy events
```

---

## Phase 8 — Kafka

Streaming real.

---

## Phase 9 — Flink

Adicionar:

```text
windows

watermarks

late events
```

---

## Phase 10 — Surf

Construir:

```text
surf conditions

surf score
```

---

## Phase 11 — API

Publicar dados.

---

## Phase 12 — BI

Criar dashboards.

---

## Phase 13 — AWS

Migrar ambiente local.

---

## Phase 14 — Terraform + CI/CD

Automatizar infraestrutura.

---

# 88. Arquitetura final desejada

```text
                             EARTHFLOW

                               SOURCES

        ┌─────────────┬──────────────┬──────────────┐
        │             │              │              │
     Weather         Ocean       Air Quality     Sensors
        │             │              │              │
        └─────────────┴───────┬──────┴──────────────┘
                              │
                         INGESTION
                              │
               ┌──────────────┴─────────────┐
               │                            │
             Batch                       Streaming
               │                            │
            Airflow                        Kafka
               │                            │
               │                           Flink
               │                            │
               └──────────────┬─────────────┘
                              ▼
                              S3

                           BRONZE
                              │
                             Spark
                              │
                           SILVER
                              │
                           ICEBERG
                              │
                   ┌──────────┴──────────┐
                   │                     │
                 Gold                Realtime
                   │                     │
          ┌────────┼────────┐            │
          │        │        │            │
       Athena   Redshift   API         Alerts
          │
          ▼
      QuickSight
```

---

# 89. Evolução futura

Depois do MVP:

```text
V2
Air Quality

V3
Earthquakes

V4
Wildfires

V5
Floods

V6
Volcanoes

V7
Storms / Hurricanes

V8
Climate Analytics

V9
Machine Learning

V10
Global Earth Data Platform
```

---

# 90. Machine Learning futuro

Use cases:

```text
weather forecasting

wave forecasting

surf quality prediction

anomaly detection

sensor failure prediction

extreme weather detection
```

Mas ML deve consumir a plataforma, não ser o centro dela.

---

# 91. Critério de sucesso técnico

O projeto será considerado maduro quando for possível executar algo como:

```text
"What was the weather and ocean condition
for this location during the last 12 months?"
```

e o fluxo for:

```text
Location
   ↓
EarthFlow API
   ↓
Gold / Iceberg
   ↓
Athena
   ↓
Weather + Ocean history
```

Enquanto, simultaneamente:

```text
Live Sensors
     ↓
Kafka
     ↓
Flink
     ↓
Realtime Metrics
```

---

# 92. North Star técnica

A principal ideia arquitetural deve ser:

> **Uma plataforma única para representar eventos da Terra através de localização + tempo + domínio.**

Ou conceitualmente:

```text
Earth Event

WHERE?
latitude / longitude

WHEN?
timestamp

WHAT?
weather / ocean / air / earthquake / fire

HOW MUCH?
measurement

FROM WHERE?
source
```

Isso permite que praticamente todos os futuros domínios ambientais se encaixem na mesma arquitetura.

---

# 93. Modelo conceitual central

```text
                    EARTH

                      │

                ┌─────┴─────┐

              SPACE        TIME

                │            │

           Location      Timestamp

                │            │

                └─────┬──────┘

                      ▼

                 Observation

                      │

          ┌───────────┼───────────┐

          ▼           ▼           ▼

       Weather      Ocean        Air

                                  │

                                  ▼

                            Earth Events
```

Esse é o ponto que torna o projeto muito maior do que simplesmente **"um projeto de clima"**.

O EarthFlow pode começar pequeno com **Weather + Ocean + Surf**, mas a arquitetura já nasce preparada para virar uma **plataforma geral de dados ambientais e geoespaciais**.

Você quer que o próximo passo seja transformar este PRD em um backlog priorizado de MVP ou detalhar primeiro os contratos de dados e schemas?
