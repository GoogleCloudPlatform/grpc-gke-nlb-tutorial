# Using Envoy Proxy to load-balance gRPC services on GKE

This repository contains the code used in the tutorial
[Using Envoy Proxy to load-balance gRPC services on GKE](https://cloud.google.com/solutions/exposing-grpc-services-on-gke-using-envoy-proxy).

This tutorial demonstrates how to expose multiple [gRPC](https://grpc.io/)
services deployed on
[Google Kubernetes Engine (GKE)](https://cloud.google.com/kubernetes-engine/)
via a single external IP address using
[Network Load Balancing](https://cloud.google.com/load-balancing/docs/network/)
and [Envoy Proxy](https://www.envoyproxy.io/). We use Envoy Proxy in this
tutorial to highlight some of the advanced features it provides for gRPC.

## Disclaimer

This is not an officially supported Google product.
