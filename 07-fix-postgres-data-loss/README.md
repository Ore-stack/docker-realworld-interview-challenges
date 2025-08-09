# Fixing Data Loss in Dockerized PostgreSQL

## Scenario:
A junior DevOps engineer set up a PostgreSQL container for local development.
Every time the container is stopped and restarted, all database data is lost.

```bash
docker-compose up -d
```

• Creates tables and inserts test data.
• Stops the container:

```bash
docker-compose down
```

• Restarts with:

```bash
docker-compose up -d
```

**• All data is gone.**

⸻

### Task for Candidate

1. Identify why the database data is disappearing.
2. Modify the configuration so data persists between container restarts.

⸻

### Expected Fix & Explanation

### Root Cause:
• Without a volume, PostgreSQL writes data inside the container’s writable layer.
• When the container is removed (docker-compose down without -v), that layer is destroyed.

⸻

### Fixed docker-compose.yml:

```yaml
version: "3.9"
services:
  db:
    image: postgres:14
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
      POSTGRES_DB: mydb
    ports:
      - "5432:5432"
    volumes:
      - db_data:/var/lib/postgresql/data

volumes:
  db_data:
```
⸻

### Why This Works:
• The named volume db_data stores database files outside the container lifecycle.
• Even if the container is removed and recreated, the data in the named volume remains intact.
