# WordPress-Nginx-CloudSQL Virtual Cluster

Create multiple WordPress sites while sharing the same external Google CloudSQL database server.

Each site is its own Kubernetes Namespace with:

One or more Pods containing:
- a WordPress/PHP-FPM container with Redis extensions
- an Nginx container with FastCGI Caching, [Pagespeed](https://www.modpagespeed.com/) extensions and [NAXSI firewall](https://github.com/nbs-system/naxsi)
- a Google CloudSQL proxy container to a private, separate database on a shared CloudSQL database server

## Requirements
An existing Kubernetes cluster with active [Core Cluster Services](https://github.com/stcox/charts/tree/master/core): Redis, LetsEncrypt and Ingress controller.
