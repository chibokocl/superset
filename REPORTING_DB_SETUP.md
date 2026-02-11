# OpenLMIS Reporting Database Setup

## Prerequisites
- Docker installed and running
- The `openlmis_export/` dump directory (ask the team for it)

## Quick Start

### 1. Start the database container

```bash
docker compose -f docker-compose.reporting.yml up -d
```

Wait a few seconds for PostgreSQL to initialize:

```bash
docker exec reporting-db pg_isready -U postgres
```

### 2. Copy the dump into the container

```bash
docker cp openlmis_export/ reporting-db:/tmp/openlmis_export
```

### 3. Create the database and restore

```bash
docker exec reporting-db psql -U postgres -c "CREATE DATABASE open_lmis_reporting;"
docker exec reporting-db pg_restore -U postgres -d open_lmis_reporting -j 8 /tmp/openlmis_export/
```

This will take several minutes depending on your machine.

### 4. Verify

```bash
docker exec reporting-db psql -U postgres -d open_lmis_reporting -c "\dt" | head -20
docker exec reporting-db psql -U postgres -d open_lmis_reporting -c "SELECT matviewname FROM pg_matviews;"
```

You should see 93+ tables and the materialized views.

### 5. Connect Superset

Add a new database connection in Superset with:

- **Database name**: `open_lmis_reporting`
- **SQLAlchemy URI**: `postgresql+psycopg2://postgres:p%40ssw0rd@localhost:5434/open_lmis_reporting`

> If connecting from another machine on the network, replace `localhost` with the host machine's IP address.

## Connection Details

| Setting          | Value                          |
|------------------|--------------------------------|
| Host             | localhost (or host machine IP) |
| Port             | 5434                           |
| Database         | open_lmis_reporting            |
| Username         | postgres                       |
| Password         | p@ssw0rd                       |

## Managing the Container

```bash
# Stop (keeps data)
docker compose -f docker-compose.reporting.yml down

# Start again (data persists)
docker compose -f docker-compose.reporting.yml up -d

# Stop and delete all data
docker compose -f docker-compose.reporting.yml down -v
```
