# Fixing Docker Multi-Container Networking Issue

## Scenario: 
You have two containers — a Python Flask API (app) and a PostgreSQL database (db) — running via docker-compose. The app fails to connect to the database with the error:

```
psycopg2.OperationalError: could not translate host name "db" to address
```

### Current docker-compose.yml: 

```yaml
version: "3.9" 
services: 
    app: 
         build: . 
         ports: 
             - "5000:5000" 
         environment: 
             - DATABASE_URL=postgresql://postgres:password@db:5432/mydb 
      db: 
           image: postgres:14 
           environment: 
               POSTGRES_USER: postgres 
               POSTGRES_PASSWORD: password 
               POSTGRES_DB: mydb
```

⸻

### Symptoms:
 • Running docker ps shows both containers are up.
 • docker exec -it app ping db → ping: db: Name or service not known.
 • The database container is healthy.

⸻

### Task for Candidate
 1. Identify the reason the Flask app cannot resolve the hostname db.
 2. Fix the configuration so the containers can communicate over Docker’s internal network.

⸻


### Why the Flask App Cannot Resolve the Hostname “db”

When you run two containers without connecting them to the same user-defined network, Docker’s built-in DNS won’t resolve one container’s name from inside the other. By default:

- Containers on Docker’s default bridge network do not get automatic name resolution  
- localhost always refers to the container itself, and “db” isn’t on that loopback  
- No user-defined network means no internal DNS entries for service names  

---

### How to Fix It

You need to place both containers on a shared, user-defined bridge network so that Docker’s embedded DNS will map the hostname db to your Postgres container’s IP.

---

## Option A: Using Docker Compose

Add an explicit network and attach both services to it:

```yaml
version: "3.9"

services:
  app:
    build: .
    ports:
      - "5000:5000"
    environment:
      - DATABASE_URL=postgresql://user:password@db:5432/mydb
    depends_on:
      - db
    networks:
      - backend

  db:
    image: postgres:14
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
      POSTGRES_DB: mydb
    networks:
      - backend

networks:
  backend:
    driver: bridge
```

### How this helps:
- Compose creates the backend network before starting services  
- The DNS entry db resolves automatically to the Postgres container  
- Within the app container, ping db or connecting via psycopg2 now works  

---

## Option B: Using Docker CLI

If you’re running containers manually:

1. Create a user-defined network
 
```bash
   docker network create backend
```

2. Start the database on that network  
   
 ```bash
  docker run -d \
     --name db \
     --network backend \
     -e POSTGRES_USER=user \
     -e POSTGRES_PASSWORD=password \
     -e POSTGRES_DB=mydb \
     postgres:14
```

3. Start your Flask app on the same network  
   
```bash
   docker run -d \
     --name app \
     --network backend \
     -p 5000:5000 \
     -e DATABASE_URL=postgresql://user:password@db:5432/mydb \
     my-flask-image
 ```
  
Now inside the **app** container:

- **ping db** will succeed  
- Your Flask app can connect to Postgres using hostname db  

---

By attaching both containers to a user-defined bridge network, you enable Docker’s internal DNS to resolve service names, letting your Flask application reach db:5432 without any hacks.
