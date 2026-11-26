# Real-time Energy Consumption Monitoring

A real-time big data pipeline that monitors and analyzes energy usage data from homes, buildings, or factories. The system collects data such as energy usage, voltage levels, device-level consumption, and renewable energy output, then identifies peak usage hours, breaks down consumption by device, and detects sudden increases in energy use.

## Architecture

```
Data Source -> Kafka (energy-raw) -> NiFi (ETL) -> Kafka (energy-clean) -> Spark Streaming -> PostgreSQL
```

| Component | Role |
|-----------|------|
| Kafka | Message broker; receives raw data and transformed data via topics |
| NiFi | Consumes raw data from Kafka and performs ETL |
| Spark Streaming | Real-time analytics and processing |
| PostgreSQL | Stores processed results |

## Analytics

- **Peak usage hours**: window aggregation to find when consumption is highest
- **Renewable vs. consumption**: comparing renewable energy output with usage
- **Device breakdown**: energy usage per device (AC, refrigerator, lights, TV, computers)
- **Voltage monitoring**: detecting readings outside the normal range
- **Anomaly detection**: spotting sudden increases in energy use

## Prerequisites

- Docker Desktop (with WSL 2 on Windows)
- Python 3.x
- Git

## Getting Started

### 1. Start the infrastructure

```bash
docker compose up -d
docker ps
```

You should see 4 running containers: `zookeeper`, `kafka`, `postgres`, `nifi`.

### 2. Create the Kafka topics

```bash
docker exec kafka kafka-topics --create --topic energy-raw --bootstrap-server kafka:29092 --partitions 3 --replication-factor 1
docker exec kafka kafka-topics --create --topic energy-clean --bootstrap-server kafka:29092 --partitions 3 --replication-factor 1
docker exec kafka kafka-topics --list --bootstrap-server kafka:29092
```

### 3. Verify PostgreSQL

```bash
docker exec -it postgres psql -U energy -d energy_db -c "SELECT 1;"
```

### 4. Open NiFi

NiFi takes 2-3 minutes to start: http://localhost:8080/nifi

## Connection Settings

| Service | From your machine (Python, Spark) | From inside Docker (NiFi) |
|---------|-----------------------------------|---------------------------|
| Kafka | `localhost:9092` | `kafka:29092` |
| PostgreSQL | `localhost:5432` | `postgres:5432` |
| NiFi UI | `http://localhost:8080/nifi` | - |

**Kafka topics**
- `energy-raw`: raw data from the producer
- `energy-clean`: data after NiFi transformation

**PostgreSQL**
- Database: `energy_db`
- User: `energy`
- Password: `energy123`

## Useful Commands

```bash
docker compose down          # stop everything (data is kept)
docker compose up -d         # start again
docker compose logs kafka    # view Kafka logs
docker compose logs nifi     # view NiFi logs
```
