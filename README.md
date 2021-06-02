# Using Envoy to load-balance gRPC services on GKE

This repository contains the code used in the tutorial
[Using Envoy Proxy to load-balance gRPC services on GKE](https://cloud.google.com/architecture/exposing-grpc-services-on-gke-using-envoy-proxy).

This tutorial demonstrates how to expose multiple [gRPC](https://grpc.io/)
services deployed on
[Google Kubernetes Engine (GKE)](https://cloud.google.com/kubernetes-engine/)
via a single external IP address using
[Network Load Balancing](https://cloud.google.com/load-balancing/docs/network/)
and [Envoy Proxy](https://www.envoyproxy.io/). The tutorial uses Envoy Proxy to
highlight some of the advanced features it provides for gRPC.

## Quick start

1.  Create a self-signed TLS certificate and private key:

    ```sh
    openssl req -x509 -newkey rsa:4096 -nodes -sha256 -days 365 \
        -keyout privkey.pem -out cert.pem -extensions san \
        -config \
        <(echo "[req]";
          echo distinguished_name=req;
          echo "[san]";
          echo subjectAltName=DNS:grpc.example.com
         ) \
        -subj '/CN=grpc.example.com'
    ```

2.  Create a Kubernetes Secret called `envoy-certs` that contains the
    self-signed TLS certificate and private key:

    ```sh
    kubectl create secret tls envoy-certs --key=privkey.pem --cert=cert.pem \
        --dry-run=client --output=yaml | kubectl apply --filename -
    ```

3.  Build the container images for the sample apps `echo-grpc` and
    `reverse-grpc`, and deploy all the resources in this repository to a
    Kubernetes cluster, using [Skaffold](https://skaffold.dev):

    ```sh
    export SKAFFOLD_DEFAULT_REPO=gcr.io/$(gcloud config get-value core/project)

    skaffold run
    ```

## Test the solution

1.  Install [`grpcurl`](https://github.com/fullstorydev/grpcurl):

    ```sh
    go install github.com/fullstorydev/grpcurl/cmd/grpcurl@latest
    ```

    If you don't have the Go distribution installed, you can instead
    [download a binary release](https://github.com/fullstorydev/grpcurl/releases/latest).

2.  Get the external IP address of the `envoy` Kubernetes Service and store it
    in an environment variable:

    ```sh
    EXTERNAL_IP=$(kubectl get service envoy \
        --output=jsonpath='{.status.loadBalancer.ingress[0].ip}')
    ```

3.  Send a request to the `echo-grpc` sample app:

    ```sh
    grpcurl -d '{"content": "echo"}' -proto echo-grpc/api/echo.proto \
        -authority grpc.example.com -cacert cert.pem -v \
        $EXTERNAL_IP:443 api.Echo/Echo
    ```

4.  Send a request to the `reverse-grpc` sample app:

    ```sh
    grpcurl -d '{"content": "reverse"}' -proto reverse-grpc/api/reverse.proto \
        -authority grpc.example.com -cacert cert.pem -v \
        $EXTERNAL_IP:443 api.Reverse/Reverse
    ```

## Cleaning up

1.  Delete the Kubernetes resources:

    ```sh
    skaffold delete

    kubectl delete secret tls envoy-certs
    ```

2.  Delete the container images from Container Registry:

    ```sh
    gcloud container images list-tags gcr.io/$GOOGLE_CLOUD_PROJECT/echo-grpc \
        --format 'value(digest)' | xargs -I {} gcloud container images delete \
        --force-delete-tags --quiet gcr.io/$GOOGLE_CLOUD_PROJECT/echo-grpc@sha256:{}

    gcloud container images list-tags gcr.io/$GOOGLE_CLOUD_PROJECT/reverse-grpc \
        --format 'value(digest)' | xargs -I {} gcloud container images delete \
        --force-delete-tags --quiet gcr.io/$GOOGLE_CLOUD_PROJECT/reverse-grpc@sha256:{}
    ```

## Disclaimer

This is not an officially supported Google product.
