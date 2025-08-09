# Debugging Networking Issues Between Containers

## Scenario:
You’re a DevOps engineer at DataStream Analytics.
Two services are containerized:
• API service (Node.js) running on port 5000
• Frontend service (React) running on port 3000

Both are defined in docker-compose.yml.
The frontend tries to call the API at http://localhost:5000/api, but you keep getting “connection refused” errors from the browser when running in Docker.

⸻

### Given docker-compose.yml:

 ```yaml
version: "3.9"

services:
  api:
    build: ./api
    ports:
      - "5000:5000"
  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
```

⸻

### Task for Candidate
1. Identify the cause of the connection issue between frontend and api.
2. Update docker-compose.yml so the containers communicate successfully within Docker’s network.
3. Explain why localhost did not work.

⸻

### Expected Solution

### Root Cause:
Inside Docker, localhost refers to the container itself, not the host machine or other containers.
The frontend container must call the API using the service name from docker-compose’s network.

⸻

### Updated docker-compose.yml:

```yaml
version: "3.9"

services:
  api:
    build: ./api
    ports:
      - "5000:5000"

  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
    environment:
      - REACT_APP_API_URL=http://api:5000/api
```

⸻

### Explanation:
• By default, docker-compose creates a bridge network where each service is reachable by its service name as a hostname.
• Using http://api:5000 allows the frontend container to connect to the API container via Docker’s internal DNS.
