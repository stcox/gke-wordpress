# WordPress - Nginx - CloudSQL for Kubernetes

Create multiple WordPress sites while sharing the same external Google CloudSQL database server.

Each site has its own Kubernetes Namespace with:

One or more Pods containing:
- a WordPress/PHP-FPM container with Redis extensions
- an Nginx container with FastCGI Caching, [Pagespeed](https://www.modpagespeed.com/) extensions and [NAXSI firewall](https://github.com/nbs-system/naxsi)
- a Google CloudSQL proxy container to a private, separate database on a shared CloudSQL database server

## Requirements
An existing Kubernetes cluster, Redis, LetsEncrypt, and an Ingress controller.

Installation and Usage
Visit USAGE.md.

Acknowledgements
This project was inspired by [daxio/k8s-lemp](https://github.com/daxio/k8s-lemp) and builds on it with the various other official Docker images and Kubernetes applications mentioned previously.
