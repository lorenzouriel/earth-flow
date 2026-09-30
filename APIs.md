Perfeito. Para o **EarthFlow**, eu separaria isso em três blocos bem definidos:

```text
1. Source APIs
   Dados externos que entram na plataforma

2. Data Contracts
   Regras que definem como cada domínio representa seus dados

3. Serving APIs
   APIs do próprio EarthFlow para consumidores externos
```

Abaixo está uma especificação técnica já pensada para virar `/schemas`, `/api` e `/contracts` no repositório.

# 1. Arquitetura de contratos

```text
                    EXTERNAL WORLD

           Weather APIs      Ocean APIs
                 │               │
                 ▼               ▼
              Adapters / Connectors
                      │
                      ▼
                SOURCE CONTRACT
                      │
                      ▼
                   BRONZE
                      │
                      ▼
                NORMALIZATION
                      │
                      ▼
              CANONICAL CONTRACT
                      │
                      ▼
                   SILVER
                      │
            ┌─────────┴─────────┐
            ▼                   ▼
          GOLD                STREAM
            │                   │
            ▼                   ▼
      Serving API             Kafka
```

A regra principal é:

> **O schema externo nunca deve virar diretamente o schema interno.**

Se uma API externa mudar:

```text
temperature_2m
```

para:

```text
air_temperature
```

isso não pode quebrar toda a plataforma.

O adapter converte ambos para:

```text
temperature_c
```

---

# 2. Estrutura de schemas

Eu criaria:

```text
schemas/
│
├── common/
│   ├── location.schema.json
│   ├── source.schema.json
│   └── metadata.schema.json
│
├── weather/
│   ├── weather-observation-v1.schema.json
│   ├── weather-hourly-v1.schema.json
│   └── weather-daily-v1.schema.json
│
├── ocean/
│   ├── ocean-observation-v1.schema.json
│   └── swell-observation-v1.schema.json
│
├── surf/
│   ├── surf-peak-v1.schema.json
│   └── surf-condition-v1.schema.json
│
├── streaming/
│   ├── event-envelope-v1.schema.json
│   ├── weather-event-v1.schema.json
│   └── ocean-event-v1.schema.json
│
└── api/
    ├── weather-current-response.json
    ├── weather-history-response.json
    └── ocean-current-response.json
```

---

# 3. Identificadores

Eu evitaria IDs incrementais como:

```text
1
2
3
```

para eventos.

Usaria:

```text
UUID / ULID
```

Exemplo:

```text
01K6P7M0X5Y5M27CE5CA4RNQ5S
```

Para entidades estáveis:

```text
location_id
station_id
sensor_id
source_id
```

podemos usar UUID.

Para eventos:

```text
event_id
```

ULID é interessante porque possui ordenação temporal.

---

# 4. Contrato base de Location

Esse é provavelmente o contrato mais importante da plataforma.

```json
{
  "location_id": "4f873b56-aabe-4f50-8ab5-412a45f8be90",
  "name": "Praia Grande",
  "location_type": "city",

  "coordinates": {
    "latitude": -24.0058,
    "longitude": -46.4028
  },

  "country": {
    "code": "BR",
    "name": "Brazil"
  },

  "region": {
    "code": "SP",
    "name": "São Paulo"
  },

  "timezone": "America/Sao_Paulo",

  "elevation_m": 12.4
}
```

## JSON Schema

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "earthflow/location/v1",
  "title": "Location",
  "type": "object",
  "required": [
    "location_id",
    "name",
    "location_type",
    "coordinates"
  ],
  "properties": {
    "location_id": {
      "type": "string",
      "format": "uuid"
    },
    "name": {
      "type": "string"
    },
    "location_type": {
      "type": "string",
      "enum": [
        "country",
        "state",
        "city",
        "beach",
        "surf_peak",
        "weather_station",
        "buoy",
        "sensor"
      ]
    },
    "coordinates": {
      "type": "object",
      "required": [
        "latitude",
        "longitude"
      ],
      "properties": {
        "latitude": {
          "type": "number",
          "minimum": -90,
          "maximum": 90
        },
        "longitude": {
          "type": "number",
          "minimum": -180,
          "maximum": 180
        }
      }
    },
    "timezone": {
      "type": "string"
    },
    "elevation_m": {
      "type": ["number", "null"]
    }
  }
}
```

---

# 5. Weather Observation Contract

Esse seria o contrato canônico de Weather.

```json
{
  "observation_id": "01K6P84YTKAK8TPDRXSYVRBWTP",

  "location_id": "4f873b56-aabe-4f50-8ab5-412a45f8be90",

  "station_id": null,

  "observation_time": "2026-09-30T12:00:00Z",

  "temperature": {
    "value": 24.3,
    "unit": "celsius"
  },

  "apparent_temperature": {
    "value": 25.1,
    "unit": "celsius"
  },

  "humidity_pct": 72,

  "pressure_hpa": 1014.2,

  "precipitation_mm": 0,

  "cloud_cover_pct": 35,

  "visibility_m": 10000,

  "wind": {
    "speed_ms": 4.8,
    "gust_ms": 7.1,
    "direction_deg": 135
  },

  "source": {
    "source_id": "weather-open-001",
    "provider": "external-weather-api"
  },

  "metadata": {
    "schema_version": "1.0",
    "ingested_at": "2026-09-30T12:01:12Z"
  }
}
```

---

# 6. Weather canonical table

No Silver eu evitaria objetos aninhados.

Teríamos uma tabela mais analítica:

```text
weather_observation
```

| Campo | Tipo |
|---|---|
| observation_id | string |
| location_id | string |
| station_id | string nullable |
| observation_time | timestamp |
| temperature_c | double |
| apparent_temperature_c | double |
| humidity_pct | double |
| pressure_hpa | double |
| precipitation_mm | double |
| rain_mm | double |
| snowfall_mm | double |
| cloud_cover_pct | double |
| visibility_m | double |
| wind_speed_ms | double |
| wind_gust_ms | double |
| wind_direction_deg | double |
| source_id | string |
| schema_version | string |
| ingested_at | timestamp |

Isso é bem melhor para Spark/Athena/Iceberg.

---

# 7. Regras do Weather Contract

### Temperature

```text
unit = Celsius

-100 <= temperature_c <= 70
```

### Humidity

```text
0 <= humidity_pct <= 100
```

### Wind direction

```text
0 <= wind_direction_deg < 360
```

### Pressure

```text
pressure_hpa > 0
```

### Precipitation

```text
precipitation_mm >= 0
```

---

# 8. Ocean Observation Contract

```json
{
  "observation_id": "01K6P876GGWK8S8BG9AP29HKXW",

  "location_id": "ocean-point-001",

  "buoy_id": "BR-SP-BUOY-001",

  "observation_time": "2026-09-30T12:00:00Z",

  "waves": {
    "height_m": 1.4,
    "period_s": 9.8,
    "direction_deg": 145
  },

  "primary_swell": {
    "height_m": 1.1,
    "period_s": 11.2,
    "direction_deg": 135
  },

  "secondary_swell": {
    "height_m": 0.4,
    "period_s": 6.2,
    "direction_deg": 195
  },

  "water_temperature_c": 21.8,

  "current": {
    "speed_ms": 0.7,
    "direction_deg": 155
  },

  "source_id": "ocean-source-001",

  "ingested_at": "2026-09-30T12:02:00Z",

  "schema_version": "1.0"
}
```

---

# 9. Ocean Silver Schema

```text
ocean_observation
```

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

secondary_swell_height_m
secondary_swell_period_s
secondary_swell_direction_deg

water_temperature_c

current_speed_ms
current_direction_deg

source_id

schema_version

ingested_at
```

---

# 10. Surf Peak Contract

```json
{
  "surf_peak_id": "surf-br-sp-pg-forte",

  "name": "Canto do Forte",

  "beach": "Praia Grande",

  "location": {
    "latitude": -24.008,
    "longitude": -46.412
  },

  "country_code": "BR",

  "state_code": "SP",

  "preferred_conditions": {
    "swell_direction_min_deg": 90,
    "swell_direction_max_deg": 160,

    "wind_direction_min_deg": 270,
    "wind_direction_max_deg": 360,

    "wave_height_min_m": 0.5,
    "wave_height_max_m": 2.0
  },

  "skill_level": [
    "beginner",
    "intermediate"
  ]
}
```

---

# 11. Surf Condition Contract

Esse será derivado de Weather + Ocean.

```json
{
  "surf_peak_id": "surf-br-sp-pg-forte",

  "observation_time": "2026-09-30T12:00:00Z",

  "wave_height_m": 1.3,

  "swell": {
    "height_m": 1.1,
    "period_s": 11.2,
    "direction_deg": 135
  },

  "wind": {
    "speed_ms": 3.2,
    "direction_deg": 300
  },

  "water_temperature_c": 21.8,

  "surf_score": 8.1,

  "condition": "good",

  "schema_version": "1.0"
}
```

---

# 12. Event Envelope

Todos os eventos Kafka deveriam seguir o mesmo envelope.

```json
{
  "event_id": "01K6P91CX7EWKATBFM74T3HMXE",

  "event_type": "weather.observation",

  "event_version": "1.0",

  "event_time": "2026-09-30T12:00:00Z",

  "produced_at": "2026-09-30T12:00:01Z",

  "producer": "weather-sensor-simulator",

  "entity": {
    "type": "weather_station",
    "id": "BR-SP-001"
  },

  "location": {
    "latitude": -23.55,
    "longitude": -46.63
  },

  "correlation_id": null,

  "payload": {}
}
```

---

# 13. Por que usar envelope?

Porque todo evento passa a ter:

```text
event_id
event_type
event_time
producer
version
entity
location
payload
```

Isso facilita:

- observabilidade;
- deduplicação;
- replay;
- DLQ;
- tracing;
- schema evolution;
- debugging.

---

# 14. Kafka topics

Eu começaria com:

```text
earthflow.weather.observation.v1

earthflow.ocean.observation.v1

earthflow.sensor.health.v1

earthflow.environment.event.v1
```

Depois:

```text
earthflow.surf.condition.v1

earthflow.air-quality.observation.v1
```

---

# 15. Partition Keys

### Weather

```text
station_id
```

ou:

```text
location_id
```

### Ocean

```text
buoy_id
```

### Surf

```text
surf_peak_id
```

Isso garante ordenação por entidade.

---

# 16. Sensor Contract

```json
{
  "sensor_id": "BR-SP-WTH-001",

  "sensor_type": "weather_station",

  "location_id": "loc-br-sp-001",

  "status": "online",

  "capabilities": [
    "temperature",
    "humidity",
    "pressure",
    "wind"
  ],

  "frequency_seconds": 10,

  "installed_at": "2026-01-01T00:00:00Z",

  "metadata": {
    "manufacturer": "earthflow-simulator",
    "model": "virtual-weather-v1"
  }
}
```

---

# 17. Sensor Health Event

```json
{
  "event_id": "01K6...",
  "event_type": "sensor.health",

  "event_time": "2026-09-30T12:00:00Z",

  "sensor_id": "BR-SP-WTH-001",

  "status": "online",

  "metrics": {
    "latency_ms": 120,
    "events_per_minute": 60,
    "invalid_readings": 0
  }
}
```

---

# 18. Data Source Contract

Cada fonte externa deve ser registrada.

```json
{
  "source_id": "weather-provider-001",

  "name": "Weather Provider",

  "domain": "weather",

  "source_type": "REST_API",

  "frequency": "hourly",

  "enabled": true,

  "capabilities": [
    "temperature",
    "humidity",
    "precipitation",
    "wind"
  ]
}
```

Tabela:

```text
data_source
```

```text
source_id

name
provider

domain

source_type

frequency

enabled

schema_version

created_at
updated_at
```

---

# 19. Ingestion Contract

Também devemos definir contrato de execução.

```json
{
  "pipeline_run_id": "run-01K6P...",

  "pipeline": "weather-api-ingestion",

  "source_id": "weather-provider-001",

  "started_at": "2026-09-30T12:00:00Z",

  "finished_at": "2026-09-30T12:01:42Z",

  "status": "SUCCESS",

  "metrics": {
    "records_read": 10000,
    "records_written": 9987,
    "records_rejected": 13
  }
}
```

---

# 20. Pipeline Run Table

```text
pipeline_run
```

Campos:

```text
pipeline_run_id

pipeline_name

source_id

started_at

finished_at

status

records_read

records_written

records_rejected

duration_ms

error_message
```

---

# 21. Bronze Contract

O Bronze não deve modificar excessivamente o payload original.

Eu armazenaria:

```json
{
  "ingestion": {
    "pipeline_run_id": "run-...",
    "source_id": "source-...",
    "ingested_at": "2026-09-30T12:00:03Z"
  },

  "request": {
    "location_id": "loc-123"
  },

  "payload": {
    "...": "original provider response"
  }
}
```

Isso é importante para replay.

---

# 22. Silver Contract

Silver será o **modelo canônico da organização**.

```text
External API A ─┐
                │
External API B ─┼──► Canonical Weather Schema
                │
Sensor Stream ──┘
```

Portanto:

```text
provider.temperature_2m

provider.temp

sensor.temperature
```

viram:

```text
temperature_c
```

---

# 23. Gold Contracts

Gold é orientado a consumo.

Exemplo:

```text
weather_hourly
```

Schema:

```text
location_id

timestamp_hour

temperature_avg_c
temperature_min_c
temperature_max_c

humidity_avg_pct

precipitation_total_mm

wind_avg_ms
wind_max_ms

observation_count
```

---

# 24. Weather Daily

```text
weather_daily
```

```text
location_id

date

temperature_min_c
temperature_max_c
temperature_avg_c

precipitation_total_mm

humidity_avg_pct

pressure_avg_hpa

wind_avg_ms
wind_max_ms

observation_count
```

---

# 25. Ocean Hourly

```text
ocean_hourly
```

```text
location_id

timestamp_hour

wave_height_avg_m
wave_height_max_m

wave_period_avg_s

swell_height_avg_m
swell_period_avg_s

water_temperature_avg_c

observation_count
```

---

# 26. Surf Conditions Gold

```text
surf_conditions
```

```text
surf_peak_id

observation_time

wave_height_m

swell_height_m
swell_period_s
swell_direction_deg

wind_speed_ms
wind_direction_deg

water_temperature_c

surf_score

condition
```

---

# 27. API do EarthFlow

Eu separaria as APIs por versão:

```text
/api/v1/
```

Estrutura:

```text
/api/v1

    /locations

    /weather

    /ocean

    /surf

    /sources
```

---

# 28. Locations API

### Listar localização

```http
GET /api/v1/locations/{location_id}
```

Response:

```json
{
  "location_id": "loc-br-sp-santos",
  "name": "Santos",
  "type": "city",

  "coordinates": {
    "latitude": -23.96,
    "longitude": -46.33
  },

  "country_code": "BR",

  "state_code": "SP"
}
```

---

# 29. Search Location

```http
GET /api/v1/locations
```

Parameters:

```text
name

country

state

type

lat

lon

radius
```

Exemplo:

```http
GET /api/v1/locations?country=BR&state=SP&type=beach
```

---

# 30. Current Weather

```http
GET /api/v1/weather/current
```

Pode receber:

```text
location_id
```

ou:

```text
lat
lon
```

Exemplo:

```http
GET /api/v1/weather/current?location_id=loc-br-sp-pg
```

---

# 31. Current Weather Response

```json
{
  "location": {
    "location_id": "loc-br-sp-pg",
    "name": "Praia Grande"
  },

  "observation_time": "2026-09-30T12:00:00Z",

  "temperature_c": 24.3,

  "apparent_temperature_c": 25.1,

  "humidity_pct": 72,

  "pressure_hpa": 1014,

  "wind": {
    "speed_ms": 4.8,
    "direction_deg": 135
  },

  "precipitation_mm": 0,

  "source": "earthflow"
}
```

---

# 32. Weather History

```http
GET /api/v1/weather/history
```

Parameters:

```text
location_id

start

end

resolution
```

Resolution:

```text
raw

hourly

daily
```

Exemplo:

```http
GET /api/v1/weather/history
    ?location_id=loc-br-sp-pg
    &start=2026-09-01T00:00:00Z
    &end=2026-09-30T23:59:59Z
    &resolution=daily
```

---

# 33. History Response

```json
{
  "location_id": "loc-br-sp-pg",

  "resolution": "daily",

  "start": "2026-09-01",

  "end": "2026-09-30",

  "data": [
    {
      "date": "2026-09-01",
      "temperature_min_c": 18.2,
      "temperature_max_c": 25.4,
      "temperature_avg_c": 21.7,
      "precipitation_mm": 2.1
    }
  ]
}
```

---

# 34. Ocean Current API

```http
GET /api/v1/ocean/current
```

Parameters:

```text
location_id

lat
lon
```

Response:

```json
{
  "location_id": "ocean-br-sp-pg",

  "observation_time": "2026-09-30T12:00:00Z",

  "wave": {
    "height_m": 1.4,
    "period_s": 9.8,
    "direction_deg": 145
  },

  "swell": {
    "height_m": 1.1,
    "period_s": 11.2,
    "direction_deg": 135
  },

  "water_temperature_c": 21.8
}
```

---

# 35. Ocean History

```http
GET /api/v1/ocean/history
```

Parâmetros:

```text
location_id

start

end

resolution
```

---

# 36. Surf Conditions API

```http
GET /api/v1/surf/conditions
```

Exemplo:

```http
GET /api/v1/surf/conditions?surf_peak_id=surf-br-sp-pg-forte
```

Response:

```json
{
  "surf_peak": {
    "id": "surf-br-sp-pg-forte",
    "name": "Canto do Forte"
  },

  "conditions": {
    "wave_height_m": 1.3,

    "swell_height_m": 1.1,

    "swell_period_s": 11.2,

    "swell_direction_deg": 135,

    "wind_speed_ms": 3.2,

    "wind_direction_deg": 300,

    "water_temperature_c": 21.8
  },

  "surf_score": 8.1,

  "condition": "good"
}
```

---

# 37. Surf Peaks API

```http
GET /api/v1/surf/peaks
```

Filters:

```text
country

state

city

lat

lon

radius
```

Exemplo:

```http
GET /api/v1/surf/peaks
    ?lat=-24.0
    &lon=-46.4
    &radius=50
```

---

# 38. Sources API

Útil principalmente para administração.

```http
GET /api/v1/sources
```

Response:

```json
[
  {
    "source_id": "weather-001",
    "domain": "weather",
    "status": "healthy",
    "last_ingestion": "2026-09-30T12:00:00Z"
  }
]
```

---

# 39. Health API

```http
GET /health
```

Response:

```json
{
  "status": "healthy",

  "dependencies": {
    "database": "healthy",
    "data_lake": "healthy",
    "kafka": "healthy"
  }
}
```

---

# 40. API Error Contract

Todos os erros devem usar um único formato.

```json
{
  "error": {
    "code": "LOCATION_NOT_FOUND",

    "message": "Location was not found.",

    "request_id": "req-0182718",

    "timestamp": "2026-09-30T12:20:00Z"
  }
}
```

---

# 41. HTTP Status padrão

```text
200
Success

400
Invalid request

404
Resource not found

409
Conflict

422
Validation error

429
Rate limit

500
Internal error

503
Dependency unavailable
```

---

# 42. Pagination Contract

Para endpoints com listas:

```http
GET /locations?page_size=100&cursor=abc123
```

Response:

```json
{
  "data": [],
  "pagination": {
    "next_cursor": "xyz123",
    "has_more": true
  }
}
```

Eu prefiro **cursor pagination** em vez de:

```text
page=50000
```

para datasets grandes.

---

# 43. Time Contract

Essa regra precisa estar documentada desde o início.

Internamente:

```text
UTC
```

Sempre.

Exemplo:

```text
2026-09-30T15:00:00Z
```

A timezone local fica como metadata:

```text
America/Sao_Paulo
```

Nunca armazenar:

```text
30/09/2026 12:00
```

como timestamp principal.

---

# 44. Units Contract

Definir um padrão global.

| Medida | Unidade |
|---|---|
| Temperature | Celsius |
| Distance | meters |
| Rain | millimeters |
| Wave height | meters |
| Wind | m/s |
| Current | m/s |
| Pressure | hPa |
| Direction | degrees |
| Time | UTC |
| Coordinates | WGS84 |

Isso elimina muita dor futura.

---

# 45. Direction Contract

Sempre:

```text
0° = North

90° = East

180° = South

270° = West
```

Range:

```text
0 <= direction < 360
```

---

# 46. Missing Values

Nunca usar:

```text
-999

9999

N/A
```

internamente.

Use:

```json
null
```

Por exemplo:

```json
{
  "wave_height_m": null
}
```

---

# 47. Data Quality Flags

Cada observação poderá receber:

```text
quality_status
```

Valores:

```text
VALID

SUSPECT

INVALID

MISSING
```

E:

```text
quality_flags
```

Exemplo:

```json
{
  "quality_status": "SUSPECT",

  "quality_flags": [
    "TEMPERATURE_SPIKE",
    "SENSOR_LATENCY_HIGH"
  ]
}
```

---

# 48. Provenance

Cada dado deveria permitir responder:

> De onde isso veio?

Por isso Silver deverá possuir:

```text
source_id

source_record_id

pipeline_run_id

ingested_at
```

Exemplo:

```text
Weather observation
        │
        ▼
source_id
        │
        ▼
Open API request
        │
        ▼
pipeline_run_id
```

---

# 49. Versionamento

Nunca simplesmente alterar:

```text
weather-observation
```

em produção.

Usaremos:

```text
weather-observation-v1

weather-observation-v2
```

Mudanças compatíveis:

```text
Adicionar campo opcional
```

podem continuar na mesma major version.

Breaking change:

```text
rename field
change type
remove field
change semantic
```

gera nova major version.

---

# 50. Exemplo de evolução

### V1

```json
{
  "temperature_c": 24
}
```

### V1.1

Compatível:

```json
{
  "temperature_c": 24,
  "humidity_pct": 72
}
```

### V2

Breaking:

```json
{
  "temperature": {
    "value": 24,
    "unit": "celsius"
  }
}
```

---

# 51. Schema Registry

Quando Kafka estiver no projeto:

```text
Producer
   │
   ▼
Schema Registry
   │
   ▼
Kafka
   │
   ▼
Consumer
```

Podemos começar com:

```text
JSON Schema
```

e posteriormente estudar:

```text
Avro

Protobuf
```

---

# 52. Organização dos Data Contracts

Eu faria uma pasta separada:

```text
contracts/
│
├── weather/
│   ├── README.md
│   ├── contract.yaml
│   └── examples/
│
├── ocean/
│
├── geography/
│
└── surf/
```

Um contrato poderia conter:

```yaml
name: weather_observation

version: 1.0.0

owner: weather-domain

primary_key:
  - observation_id

business_key:
  - location_id
  - observation_time
  - source_id

freshness:
  expected: 1h

fields:

  temperature_c:
    type: double
    nullable: true
    unit: celsius

  humidity_pct:
    type: double
    nullable: true
    minimum: 0
    maximum: 100
```

---

# 53. Contract ownership

Cada domínio deve ser dono dos próprios schemas.

```text
Weather Team
    owns
weather_observation

Ocean Team
    owns
ocean_observation

Surf
    owns
surf_condition
```

Mesmo sendo um projeto pessoal, vale usar esse padrão.

Ele ensina **Data Mesh/Data Contracts** sem precisar transformar a arquitetura em Data Mesh.

---

# 54. Contratos do MVP

Eu consideraria obrigatórios estes contratos:

```text
location.v1

data-source.v1

pipeline-run.v1

weather-observation.v1

weather-hourly.v1

weather-daily.v1

ocean-observation.v1

ocean-hourly.v1

surf-peak.v1

surf-condition.v1

event-envelope.v1

sensor.v1

sensor-health.v1
```

Esse é um conjunto suficientemente pequeno para começar e suficientemente robusto para crescer.

---

# 55. Banco relacional de metadata

Eu usaria PostgreSQL para informações administrativas:

```text
location

weather_station

ocean_buoy

surf_peak

data_source

pipeline_run
```

E **não** para os bilhões de observations.

Observations ficam no:

```text
S3
+
Iceberg
```

---

# 56. Separação final

A arquitetura de dados fica:

```text
                    PostgreSQL

          metadata / configuration

    location
    source
    sensor
    buoy
    surf_peak
    pipeline_run

------------------------------------------------

                     Iceberg

              analytical datasets

    weather_observation
    ocean_observation
    weather_hourly
    weather_daily
    surf_conditions

------------------------------------------------

                      Kafka

                realtime events

    weather.observation
    ocean.observation
    sensor.health
```

Essa separação é importante.

---

# 57. Estrutura que eu colocaria no GitHub

```text
earthflow/
│
├── contracts/
│   ├── geography/
│   ├── weather/
│   ├── ocean/
│   └── surf/
│
├── schemas/
│   ├── json/
│   ├── avro/
│   └── iceberg/
│
├── api/
│   ├── routes/
│   ├── models/
│   ├── services/
│   └── openapi/
│
├── ingestion/
│
├── streaming/
│
├── processing/
│
└── tests/
```

---

# 58. Fluxo completo usando esses contratos

Finalmente teríamos:

```text
External Weather API
        │
        ▼
    Source Adapter
        │
        ▼
    Raw Provider Schema
        │
        ▼
       Bronze
        │
        ▼
   Validation / Mapping
        │
        ▼
Weather Observation V1
        │
        ▼
       Silver
        │
        ▼
      Iceberg
        │
     ┌──┴───────────────┐
     │                  │
     ▼                  ▼
   Spark               Flink
     │                  │
     ▼                  ▼
Weather Daily      Realtime metrics
     │
     ▼
    Gold
     │
     ▼
EarthFlow API

GET /api/v1/weather/current
GET /api/v1/weather/history
```

## Ordem de implementação

Para evitar overengineering, eu implementaria exatamente nesta sequência:

1. `location.v1`
2. `data-source.v1`
3. `weather-observation.v1`
4. Bronze weather schema
5. Silver weather schema
6. `weather-hourly.v1`
7. `weather-daily.v1`
8. `ocean-observation.v1`
9. `surf-peak.v1`
10. `surf-condition.v1`
11. `event-envelope.v1`
12. Sensor schemas
13. Kafka contracts
14. REST API `/weather`
15. REST API `/ocean`
16. REST API `/surf`
17. Schema Registry

Com isso, o EarthFlow já nasce com uma fronteira muito clara entre **source schema → canonical schema → analytical schema → API contract**, que é exatamente o tipo de separação que evita que uma Data Platform grande vire um conjunto de pipelines fortemente acoplados.

Você quer que eu transforme esses contratos em arquivos JSON Schema e OpenAPI prontos para colocar no repositório?
