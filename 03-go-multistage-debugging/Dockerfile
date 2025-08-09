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
