ARG VARIANT=1.14-alpine
FROM golang:${VARIANT}

# Download the dlv (delve) debugger for go (you can comment this out if unused)
RUN apk add git && go get -u -v github.com/go-delve/delve/cmd/dlv && rm -rf /go/src

# If you want to use the debugger, you need to modify the entrypoint to the
# container and point it to the "dlv debug" command:
#   * UNCOMMENT the following ENTRYPOINT statement,
#   * COMMENT OUT the last ENTRYPOINT statement
# Start the "dlv debug" server on port 3000 of the container.
ENTRYPOINT ["dlv", "exec", "/app", "--continue", "--accept-multiclient", "--api-version=2", "--headless", "--listen=:3000", "--log"]

# If you want to run WITHOUT the debugging server:
#   * COMMENT OUT the previous ENTRYPOINT statements,
#   * UNCOMMENT the following ENTRYPOINT statement.
# ENTRYPOINT ["/app"]
