# syntax = docker/dockerfile:experimental
FROM golang:1.22.0 as builder

ARG version=dev

WORKDIR /workspace

COPY go.mod go.sum ./

RUN --mount=type=cache,target=/go/pkg/mod \
    go mod download

COPY controllers/api controllers/api
COPY controllers/config/config.go controllers/config/config.go
COPY controllers/controllers controllers/controllers
COPY controllers/cleanup controllers/cleanup
COPY controllers/coordination controllers/coordination
COPY controllers/main.go controllers/main.go
COPY controllers/webhooks controllers/webhooks

COPY kpack-image-builder/controllers/ kpack-image-builder/controllers

COPY job-task-runner/api job-task-runner/api
COPY job-task-runner/controllers job-task-runner/controllers

COPY statefulset-runner/api/ statefulset-runner/api
COPY statefulset-runner/controllers/ statefulset-runner/controllers

COPY tools tools
COPY version version

RUN --mount=type=cache,target=/root/.cache/go-build \
    --mount=type=cache,target=/go/pkg/mod \
    CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -ldflags "-X code.cloudfoundry.org/korifi/version.Version=${version}" -gcflags=all="-N -l" -o manager controllers/main.go

# Get Delve from a GOPATH not from a Go Modules project
WORKDIR /go/src/
RUN go install github.com/go-delve/delve/cmd/dlv@latest

FROM ubuntu

WORKDIR /
COPY --from=builder /workspace/manager .
COPY --from=builder /go/bin/dlv .
EXPOSE 8080 8081 9443 40000

CMD ["/dlv", "--listen=:40000", "--headless=true", "--api-version=2", "exec", "/manager", "--continue", "--accept-multiclient"]
