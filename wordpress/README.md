# WordPress-Nginx Cluster for Google CloudSQL

Create multiple WordPress sites while sharing same external Google CloudSQL database server.

Each site is a Kubernetes pod with:

1. a dedicated WordPress/PHP-FPM container with Redis extensions
2. a dedicated Nginx container with FastCGI Caching, Pagespeed extensions and NAXSI firewall
3. a proxy container to a private, separate database on a shared CloudSQL database server
4. a virtual cluster namespace

Requires [Core Cluster](https://github.com/stcox/charts/tree/master/core) chart for basic core services: Redis, LetsEncrypt and Ingress.
