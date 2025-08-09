# Ensuring Dependent Containers Start in the Correct Order

## Scenario:
You’re deploying a Django app with PostgreSQL using docker-compose.
Currently, the Django container sometimes fails to start with the error:

```
psycopg2.OperationalError: could not connect to server: Connection refused
This happens because Django tries to connect to Postgres before it’s ready.
```

⸻

### Given docker-compose.yml:

```yaml
version: "3.9"

services:
  db:
    image: postgres:14
    environment:
      POSTGRES_DB: mydb
      POSTGRES_USER: myuser
      POSTGRES_PASSWORD: mypass
    ports:
      - "5432:5432"

  web:
    build: ./web
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgres://myuser:mypass@db:5432/mydb
    depends_on:
      - db
```

⸻

### Task for Candidate
1. Explain why depends_on does not guarantee the database is ready.
2. Add a health check for the database container.
3. Modify depends_on so the web service waits until the DB is healthy.

⸻

### Expected Solution

### Root Cause:
depends_on only controls container startup order, not readiness.
The DB may take a few seconds to accept connections.

⸻

### Updated docker-compose.yml:

```yaml
version: "3.9"

services:
  db:
    image: postgres:14
    environment:
      POSTGRES_DB: mydb
      POSTGRES_USER: myuser
      POSTGRES_PASSWORD: mypass
    ports:
      - "5432:5432"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U myuser -d mydb"]
      interval: 5s
      retries: 5

  web:
    build: ./web
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgres://myuser:mypass@db:5432/mydb
    depends_on:
      db:
        condition: service_healthy
```

⸻

### Explanation:
• pg_isready checks if Postgres is accepting connections.
• The health check keeps retrying until the DB is ready.
• depends_on with condition: service_healthy ensures the web service only starts after the DB passes its health check.
