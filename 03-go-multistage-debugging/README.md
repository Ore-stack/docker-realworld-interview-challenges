# Debugging a Broken Multi-Stage Docker Build for a Go Application

## Scenario: 
A Go API project uses Docker multi-stage builds to create a lightweight production image. When deploying the container, it fails with: 

```
standard_init_linux.go:228: exec user process caused: no such file or directory 
```

## The current Dockerfile is: 

```dockerfile
# Build Stage 
FROM golang:1.20 as builder 
WORKDIR /app 
COPY . . 
RUN go build -o main . 

# Production Stage 
FROM alpine:latest 
WORKDIR /app 
COPY --from=builder /app/main . 
CMD ["./main"]
```

### Task for Candidate 
• Identify why the container fails to run. 
• Update the Dockerfile so the app runs in the final Alpine image.

### What’s Happening
1. Builder stage uses golang:1.20 (glibc-based).
2. You run go build -o main . without disabling CGO, so Go may produce a dynamically linked binary.
3. Final stage is alpine:latest (musl-based), so when Docker tries to exec ./main, the kernel can’t find the glibc loader and throws “no such file or directory.”

### Fixed Dockerfile (Static Binary + Multi-Stage):

```dockerfile
# Build Stage
FROM golang:1.20 as builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
# Build a statically linked binary
RUN CGO_ENABLED=0 GOOS=linux go build -o main .

# Production Stage
FROM alpine:latest
WORKDIR /app
COPY --from=builder /app/main .
# Make sure binary is executable
RUN chmod +x ./main
CMD ["./main"]
```

## Why This Works:
 • CGO_ENABLED=0 and GOOS=linux ensure the binary is fully static so it runs on Alpine without needing extra libs.
 • We separate go.mod and go.sum copying to leverage Docker layer caching.
 • Ensuring the binary is executable avoids permission issues.
