# nginx-vector-clickhouse

Pipeline for ingesting **nginx** access logs with **Vector** and shipping them to **ClickHouse**.

## Overview

- **Vector** reads nginx `access.log` from the host, parses entries with a custom regex, and forwards structured events to ClickHouse.
- Log format is assumed to include: `ip`, `time`, `channel`, `status`, `bytes`, `request_time`.

## Prerequisites

- Docker and Docker Compose
- Nginx writing access logs to `/var/log/nginx/access.log` on the host
- A reachable ClickHouse instance (configure endpoint and auth in `vector.toml`)

## Configuration

1. **Vector** (`vector.toml`):
   - `sources.nginx`: file source for `/var/log/nginx/access.log`, `read_from = "end"`
   - `transforms.parse`: remap that parses the log line and normalizes fields
   - `sinks.clickhouse`: endpoint, database, table, and basic auth

2. **ClickHouse**: Update `vector.toml` with your ClickHouse URL and credentials:
   - `endpoint` (e.g. `http://clickhouse.example`)
   - `auth.user` / `auth.password`

3. **Docker Compose** (`docker-compose.yaml`): mounts `vector.toml` and `/var/log/nginx` into the Vector container.

## Usage

```bash
docker compose up -d
```

Vector will tail nginx access logs and stream parsed events to ClickHouse. Ensure the ClickHouse `streaming.nginx_logs` table exists and matches the schema produced by Vector (e.g. `ip`, `server_name`, `channel`, `status`, `bytes`, `request_time`, `event_time`).

## Files

| File               | Purpose                                      |
|--------------------|----------------------------------------------|
| `docker-compose.yaml` | Runs Vector with config and log volume   |
| `vector.toml`      | Vector sources, transform, and ClickHouse sink |
