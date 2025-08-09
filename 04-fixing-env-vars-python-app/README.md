# Fixing Environment Variable Issues in a Dockerized Python App

## Scenario:
A Python Flask application uses environment variables for database credentials and configuration.
When the container runs, it crashes with:

KeyError: 'DB_HOST'

The current **Dockerfile** is:

```dockerfile
FROM python:3.11
WORKDIR /app
COPY . .
RUN pip install -r requirements.txt
CMD ["python", "app.py"]
```

The **docker-compose.yml** is:

```yaml
version: "3"
services:
  web:
    build: .
    ports:
      - "5000:5000"
```

### Task for Candidate
 • Identify why the app can’t find DB_HOST.
 • Update both Dockerfile and docker-compose.yml so that environment variables are passed correctly and accessible at runtime.

⸻

### Expected Fix & Explanation

### Root Cause:
 • The application expects DB_HOST (and likely other vars) to be available in the container’s environment.
 • The current docker-compose.yml does not define any environment variables, so the container launches without them.
 • In Python, os.environ["DB_HOST"] will throw a KeyError if it’s not set.

⸻

### Fixed Dockerfile:
**(No need to bake secrets into the image — keep it generic)*

```dockerfile
FROM python:3.11
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
EXPOSE 5000
CMD ["python", "app.py"]
```

### Fixed docker-compose.yml:

```yaml
version: "3"
services:
  web:
    build: .
    ports:
      - "5000:5000"
    environment:
      DB_HOST: mydb.example.com
      DB_USER: admin
      DB_PASS: supersecret
```

### Why This Works:
 • Environment variables are now defined in docker-compose.yml and injected into the container at runtime.
 • The Dockerfile remains generic, so no secrets are stored in the image.
 • Using environment: in Compose is the most common way to pass config to containers.

⸻

### Extra Security Note for Interviews:
 • Recommend moving credentials to a .env file and referencing them in docker-compose.yml like:

env_file:
  - .env

And .env could contain:

DB_HOST=mydb.example.com
DB_USER=admin
DB_PASS=supersecret
