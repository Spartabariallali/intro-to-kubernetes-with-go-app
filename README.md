## Deploying a Hello Go app in Kubernetes

### 1. Creating a container for Go app

- we begin by creating a container for our GO app
- This is to ensure the app is able to run on a container before being deployed to a kubernetes cluster
- In order to do this we have to create a ``` Dockerfile```

```
ROM golang:1-alpine as build

WORKDIR /app

COPY cmd cmd

RUN go build cmd/hello/hello.go

FROM alpine:latest

WORKDIR /app

COPY --from=build /app/hello /app/hello

EXPOSE 8100

ENTRYPOINT ["./hello"]
```

- we create a multi-stage build as the app is only small and doesnt require much for it to operate
- To build the container from the `Dockerfile` we run the following command:

```docker build -t hello-go
```
- To run the container and expose the internal port to your localhost, run the command:

``` docker run --name hello-g0 --rm -p 8180:8180 hello-go
```
