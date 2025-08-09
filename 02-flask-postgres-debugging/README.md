# Debugging a Broken Multi-Container Flask + Postgres Setup

## Scenario: 
You have a Flask API and a Postgres database defined in docker-compose.yml. When running docker-compose up, the Flask app container fails with: 

```
psycopg2.OperationalError: could not connect to server: Connection refused
```

 ```yaml
version: "3.9" 
services: 
  web: 
    build: . 
    ports: 
       - "5000:5000" 
     environment: 
       - DATABASE_URL=postgresql://user:password@localhost:5432/mydb 
    depends_on: - db 
db: 
   image: postgres:14 
   environment: 
     POSTGRES_USER: user 
     POSTGRES_PASSWORD: password
     POSTGRES_DB: mydb 
   ports: - "5432:5432"
```

• Identify why the Flask container cannot connect to Postgres. 
• Update the docker-compose.yml and/or application configuration so the connection works.



### Why the Flask Container Can’t Reach Postgres

Your Flask app is trying to connect to localhost:5432. Inside Docker, localhost refers to the Flask container itself, not the Postgres container. Since there’s no Postgres listener inside the web container, you get a connection refused error.
Docker Compose services live on a common network and resolve each other by service name. To reach your database, your app needs to use the db hostname that Compose assigns, not localhost.

### Minimal Changes to Fix the Connection
Change the DATABASE_URL host from localhost to db.
(Optional) Add a simple healthcheck so the Flask service waits until Postgres is ready.

Here’s an updated docker-compose.yml snippet:

```yaml
version: "3.9"

services:
  web:
    build: .
    ports:
      - "5000:5000"
    environment:
      - DATABASE_URL=postgresql://user:password@db:5432/mydb
    depends_on:
      db:
        condition: service_healthy

  db:
    image: postgres:14
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
      POSTGRES_DB: mydb
    ports:
      - "5432:5432"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user"]
      interval: 10s
      timeout: 5s
      retries: 5
```
