No longer maintained - 

# GKE WordPress with Google Cloud SQL

Deploy WordPress sites to [Google Kubernetes Engine (GKE)](https://cloud.google.com/kubernetes-engine/docs/concepts/kubernetes-engine-overview) clusters via [Helm charts](https://helm.sh/). Use as your own personal WordPress farm or as a backend to your own cloud hosting company.

 [![Donate](https://img.shields.io/badge/Donate-PayPal-green.svg)](https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=FNLE7XYVKHSS2)

GKE WordPress supports/requires:
- [Google Kubernetes Engine](https://cloud.google.com/kubernetes-engine "Google Kubernetes Engine")
- [Google Compute Engine](https://cloud.google.com/compute "Google Compute Engine")
- [Google Cloud SQL](https://cloud.google.com/sql/ "Google Cloud SQL")
- [Helm, the Kubernetes Package Manager](https://helm.sh/)

## Installation and Usage
See [**Installation and Usage**](USAGE.md) for instructions on getting up and running.
Visit [USAGE.md](USAGE.md "Installation & Usage").



## How It Works
* **WordPress**
  * Each [WordPress application server image](https://github.com/stcox/wordpress "WordPress for Kubernetes WordPress") is based on the [wordpress:php7.2-fpm](https://hub.docker.com/r/_/wordpress/ "Official WordPress Docker image") docker image with `redis` extension.
  * Each WordPress `Deployment` gets it's own `PersistentVolume` as well as `Secret` objects for storing sensitive information such as passwords for their DBs.
  * `ConfigMap`s are used to inject various `php.ini` settings for PHP 7.2.

* **NGINX**
  * Each [NGINX web server image](https://github.com/stcox/nginx) is based on the official [`Nginx`](https://hub.docker.com/_/nginx/) docker image, and comes with:
  * NBS System's [NAXSI module](https://github.com/nbs-system/naxsi). NAXSI means [NGINX](http://nginx.org/) Anti-[XSS](https://www.owasp.org/index.php/Cross-site_Scripting_%28XSS%29) & [SQL Injection](https://www.owasp.org/index.php/SQL_injection).
  * Handy configurations for NGINX and the NAXSI web application firewall are also included via `ConfigMap`s.
  * The NGINX container has multiple handy configurations for multi-site and caching, all easily deployed using `ConfigMap` objects.

* **Cloud SQL**
  * The WordPress sites all interface with one [Google Cloud SQL](https://cloud.google.com/sql/) instance, so anyone can start off with a full-fledged web farm and bring up any number of websites using a single Cloud SQL server instance and a separate database for each site.

* **Redis**
  * To reduce DB hits, the WP image is built with a `redis` PHP extension that connects to a cluster-wide Redis `Deployment`. WP must be configured to use Redis upon initializing a new WP site by installing and configuring the WP [Redis Object Cache](https://wordpress.org/plugins/redis-cache/ "Redis Object Cache plugin for WordPress") plugin.

* **Ingress/Kube Lego**
  * Websites are reached externally via an `nginx-ingress` controller. See Kubernetes documentation regarding `Ingress` in the [official docs](https://kubernetes.io/docs/user-guide/ingress/ "Ingress Resources") and on [GitHub](https://github.com/kubernetes/ingress/blob/master/controllers/nginx/README.md "NGINX Ingress Controller").
  * TLS/SSL is terminated at `nginx-ingress` via free Let's Encrypt certificates, good for all domains on your cluster. Additionally, certificate issuance is handled automatically with the [`kube-lego`](https://github.com/jetstack/kube-lego "Kube Lego").

![GKE WordPress](https://github.com/stcox/gke-wordpress/blob/master/gke-wp-diagram.png?raw=true)

## Acknowledgements
This project was inspired by and based on [daxio/k8s-lemp](https://github.com/daxio/k8s-lemp "Kubernetes LEMP Stack") and builds on it with the various other official Docker images and Kubernetes applications mentioned previously.

## Donate
[![Donate](https://img.shields.io/badge/Donate-PayPal-green.svg)](https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=FNLE7XYVKHSS2)
