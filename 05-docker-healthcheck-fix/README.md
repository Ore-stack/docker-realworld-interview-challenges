### Fixing a Docker Healthcheck Failure in a Microservice

## Scenario: 

You have a small Go-based microservice that should respond on /health at port 8080. The container builds and runs fine, but in Kubernetes and Docker Compose, it keeps restarting because the healthcheck fails. 

Here’s the current **Dockerfile:** 

```dockerfile
FROM golang:1.21 as builder 
WORKDIR /app 
COPY . . 
RUN go build -o myservice 

FROM alpine:3.19 
WORKDIR /app 
COPY --from=builder /app/myservice . 
EXPOSE 8080 
HEALTHCHECK CMD curl http://localhost:8080/health || exit 1 CMD ["./myservice"]
```

**Symptoms during docker ps:**

 ```
STATUS: unhealthy
```

**Logs inside container:**

```
sh: curl: not found
```

### Task for Candidate 

1. Identify the reason for the healthcheck failure.
2. Fix the Dockerfile so the healthcheck works without bloating the final image unnecessarily.



### Why the Healthcheck Is Failing

Inside your Alpine-based image, curl doesn’t exist by default. When Docker runs:

```
HEALTHCHECK CMD curl http://localhost:8080/health || exit 1
```

it outputs:

```
sh: curl: not found
```

so the healthcheck always fails, marking the container **unhealthy.**

---

### Fixing the Dockerfile

You need a lightweight HTTP client in the final image. Two common approaches:

1. **Use Alpine’s built-in wget** (BusyBox applet)  
2. **Install a minimal package** (`wget` or curl`) with `apk add --no-cache

Below is a minimal change that installs `wget`—about a 200 KB addition—and rewrites the healthcheck to use it.

```dockerfile
# Build Stage
FROM golang:1.21 AS builder

WORKDIR /app
COPY . .
RUN go build -o myservice

# Production Stage
FROM alpine:3.19

# Install wget for healthchecks (small footprint)
RUN apk add --no-cache wget

WORKDIR /app
COPY --from=builder /app/myservice .

EXPOSE 8080

# Use wget for the healthcheck
HEALTHCHECK --interval=30s --timeout=5s --start-period=5s \
  CMD wget -qO- http://localhost:8080/health > /dev/null 2>&1 || exit 1

CMD ["./myservice"]
```

### Key Points

- apk add --no-cache wget  
  Installs only the wget binary and its dependencies, skipping cache to keep the image lean.

- HEALTHCHECK … CMD wget -qO- …  
  Uses wget in “quiet” mode to fetch /health, discarding output and returning a proper exit code.

- Tuning parameters (`--interval`, --timeout, `--start-period`)  
  Give your service a grace period on startup and control how often checks run.

With this change, your container’s healthcheck will succeed as soon as your Go service responds on port 8080.
