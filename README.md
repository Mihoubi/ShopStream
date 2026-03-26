# ShopStream — Mini Data Engineering Pipeline (Kafka → HDFS)

## Overview
ShopStream is a mini data engineering pipeline designed for learning and demonstration. It simulates an event-streaming architecture where:

- A Kafka Producer reads a newline-delimited JSON dataset and publishes events to a Kafka topic.
- A Kafka Consumer subscribes to the topic and writes each event into HDFS as a JSON file.
- The whole stack runs locally using Docker Compose (Kafka, Zookeeper, Kafka UI, Hadoop NameNode/DataNode, YARN components, and the Python apps).

## Architecture
### Data flow
- Source dataset: `docker/producer/shopstream-data.json` (one JSON event per line)
- Kafka topic: `shopstreamtopic`
- Sink storage: HDFS directory `/data` (each message saved as a separate `*.json` file)

### Services
- Kafka stack:
  - `zookeeper` (required by this Kafka version)
  - `kafka` (broker)
  - `kafka-ui` (UI for topics/messages)
- Hadoop stack:
  - `namenode` (HDFS metadata + UI)
  - `datanode` (stores file blocks)
  - `resourcemanager`, `nodemanager` (YARN, included for completeness)
- Custom apps:
  - `producer` (`docker/producer/kafka_producer.py`)
  - `consumer` (`docker/consumer/kafka_consumer.py`)

## Prerequisites
- Docker Desktop (with Docker Compose)
- (Optional) Git

## Important note about the Docker network
In `docker-compose.yml`, the network is configured as:

```yaml
networks:
  kafka-network:
    external: true
```

So you must create it once before starting the stack:

```bash
docker network create kafka-network
```

## Quick Start
From the repo root:

1) Build and start everything:
```bash
docker compose up --build
```

2) Watch logs (optional):
```bash
docker compose logs -f producer
docker compose logs -f consumer
```

3) Stop everything:
```bash
docker compose down
```

## UIs / Useful Endpoints
- Kafka UI: http://localhost:8080
- HDFS NameNode UI: http://localhost:9870
- YARN ResourceManager UI: http://localhost:8088

## How it works (code behavior)
### Producer (`docker/producer/kafka_producer.py`)
- Waits ~40 seconds for Kafka to be available (`time.sleep(40)`)
- Connects to Kafka at `kafka:29092`
- Reads `docker/producer/shopstream-data.json`
- Publishes each parsed JSON object to topic `shopstreamtopic`

### Consumer (`docker/consumer/kafka_consumer.py`)
- Waits ~50 seconds for Kafka to be available (`time.sleep(50)`)
- Connects to Kafka at `kafka:29092`
- Subscribes to topic `shopstreamtopic` with `group_id='my-group'`
- Writes each message into HDFS via `hdfs.write_to_hdfs(...)` (stored under `/data`)

## Verifying ingestion in HDFS
After the pipeline runs, open the NameNode UI:

- http://localhost:9870
- Browse the HDFS filesystem and look for the `/data` directory.
- You should see many JSON files (one per consumed event).

## Repository structure (high-level)
- `docker-compose.yml` — orchestrates all services
- `docker/producer/` — producer Dockerfile + Python producer + dataset
- `docker/consumer/` — consumer Dockerfile + Python consumer (+ HDFS helper module)
- `COURSE.md` — detailed explanation/lesson notes

## Troubleshooting
### 1) `kafka-network` not found
Create it:
```bash
docker network create kafka-network
```

### 2) Producer/consumer fails to connect to Kafka
The apps rely on fixed sleeps (`40s` / `50s`). On slower machines Kafka may take longer.
- Re-run with longer sleeps, or
- Replace sleeps with a retry loop / healthcheck approach.

### 3) Ports already in use
This project exposes:
- `8080` (Kafka UI)
- `9870` (NameNode UI)
- `8088` (YARN UI)
- `9092` and `29092` (Kafka)

Stop conflicting services or change the host ports in `docker-compose.yml`.

## License
See `LICENSE`.
