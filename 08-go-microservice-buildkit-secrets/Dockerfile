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
