# Docker, Kubernetes and GCP Workshop

## Prerequisites
- Docker
- [GCP Account](https://console.cloud.google.com/)

## Create Go Application (main.go)

```
package main

import (
        "fmt"
        "log"
        "net/http"
        "os"
)

func main() {
        // use PORT environment variable, or default to 8080
        port := "8080"
        if fromEnv := os.Getenv("PORT"); fromEnv != "" {
                port = fromEnv
        }

        // register hello function to handle all requests
        server := http.NewServeMux()
        server.HandleFunc("/", hello)

        // start the web server on port and accept requests
        log.Printf("Server listening on port %s", port)
        err := http.ListenAndServe(":"+port, server)
        log.Fatal(err)
}

// hello responds to the request with a plain-text "Hello, world" message.
func hello(w http.ResponseWriter, r *http.Request) {
        log.Printf("Serving request: %s", r.URL.Path)
        host, _ := os.Hostname()
        fmt.Fprintf(w, "Hello, world!\n")
        fmt.Fprintf(w, "Version: 1.0.0\n")
        fmt.Fprintf(w, "Hostname: %s\n", host)
}
```

## Create a Dockerfile

```
FROM golang:1.8-alpine
ADD . /go/src/hello-app
RUN go install hello-app

FROM alpine:latest
COPY --from=0 /go/bin/hello-app .
ENV PORT 8080
CMD ["./hello-app"]
```

## Build Docker Image
```
docker build -t christianhxc/hello-go:1.0 .
```

## Run App Locally
```
docker run -d -p 8585:8080 christianhxc/hello-go:1.0
```

## Push Docker Image
```
docker login
docker push christianhxc/hello-go:1.0
```

## Kubernetes in GCP

1. Go to [Kubernetes Engin page](https://console.cloud.google.com/projectselector/kubernetes?_ga=2.191387946.-1453821167.1522093243)
2. Create a project
3. Open Cloud Shell

```
gcloud config set project [PROJECT_ID]
gcloud config set compute/zone [COMPUTE_ZONE]
```

[COMPUTE_ZONE] = us-west1-a

```
gcloud container clusters create [CLUSTER_NAME]
```

```
gcloud container clusters get-credentials [CLUSTER_NAME]
```

## Deploy to Kubernetes

```
kubectl run hello-server --image christianhxc/hello-go:1.0 --port 8080
```

```
kubectl expose deployment hello-server --type "LoadBalancer"
```

## Verify Deployment
```
kubectl get service hello-server
```

```
http://[EXTERNAL_IP]:8080
```

## Scale Out

```
kubectl scale --replicas=3 deployment/hello-server
```

## Clean Up

```
kubectl delete service hello-server
```

```
gcloud container clusters delete [CLUSTER_NAME]
```