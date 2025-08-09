# Multi-Stage Docker Build with Secrets for a Go Microservice

## Scenario:
Your team at FinServe Inc. is building a Go microservice that connects to a payment API.
The API requires a private key during build to run integration tests, but the final production image must not contain the key.

⸻

## Requirements:
1. Use multi-stage Docker builds to:
 • Compile the Go binary in a build container.
 • Copy only the final binary into a small production image.
2. Use Docker BuildKit secrets to inject the API private key only at build time (not in final image).
3. The final image should:
 • Be based on alpine:3.19.
 • Contain only the compiled binary.
 • Expose port 8080.
 • Run the binary on container start.

⸻

### Given:
• Source code in the current directory.
• Private key stored locally at /home/ubuntu/api-private-key.pem.

⸻

### Task for Candidate

Write a secure multi-stage Dockerfile that meets the above requirements.

⸻

### Expected Solution:

```dockerfile
# syntax=docker/dockerfile:1.4

### Stage 1 — Build
FROM golang:1.22-alpine AS builder

# Install build tools
RUN apk add --no-cache git

WORKDIR /app

# Copy go.mod and go.sum first for caching
COPY go.mod go.sum ./
RUN go mod download

# Copy source code
COPY . .

# Use BuildKit secrets for private key during tests
# The key will be available at /run/secrets/api_key during build
RUN --mount=type=secret,id=api_key \
    go test ./... -v && \
    go build -o app

---

### Stage 2 — Final Image
FROM alpine:3.19

WORKDIR /root/

# Copy only the binary
COPY --from=builder /app/app .

EXPOSE 8080

CMD ["./app"]
```

⸻

### Build Command with Secret:

```bash
DOCKER_BUILDKIT=1 docker build \
  --secret id=api_key,src=/home/ubuntu/api-private-key.pem \
  -t finserve-microservice .
```

⸻

### Why This Works:
• Multi-stage build: Keeps the final image small and clean by copying only the binary.
• Docker BuildKit secrets: Private key is never written into an image layer; it’s available only during build/test.
• Alpine base image: Reduces final image size and attack surface.
