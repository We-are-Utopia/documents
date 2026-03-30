# syntax=docker/dockerfile:1.7
FROM golang:1.23-alpine AS build
RUN apk add --no-cache ca-certificates tzdata git
WORKDIR /src
COPY go.mod go.sum ./
RUN --mount=type=cache,target=/go/pkg/mod go mod download
COPY . .
RUN --mount=type=cache,target=/go/pkg/mod \
    --mount=type=cache,target=/root/.cache/go-build \
    CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -trimpath -ldflags="-s -w" -o /out/app ./cmd/server

FROM gcr.io/distroless/static-debian12:nonroot AS runtime
WORKDIR /
COPY --from=build /out/app /app
EXPOSE 8080
ENTRYPOINT ["/app"]
