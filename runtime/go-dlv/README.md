# Containers for Develop

Source Repository - https://github.com/norseto/dev-containers


## Containers for Debugging
### Golang debugger(runtime/go-dlv)
This image already defined ENTRYPOINT that runs `/app`. So you should use this image like:
```
FROM norseto/go-dlv:1.14

WORKDIR /src/hello-world

# Install dependencies in go.mod and go.sum
COPY go.mod go.sum ./
RUN go mod download

# Copy rest of the application source code
COPY . ./

# Compile the application to /app.
RUN go build -o /app -v ./cmd/hello-world
```
Or override ENTRYPOINT.

#### Repository and tags
|Repository|Description|
|-|-|
|norseto/go-dlv:1.14|Golang 1.14 with delve|
|norseto/go-dlv:1.15|Golang 1.15 with delve|
