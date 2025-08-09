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
