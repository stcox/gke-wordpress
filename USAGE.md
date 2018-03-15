# WordPress-Nginx-CloudSQL Kubernetes Helm Charts
Host multiple WordPress sites on Compute Engine and Google Cloud SQL.
Each site has its own Kubernetes Namespace with one or more Pods containing:
- a WordPress/PHP-FPM container with Redis extensions
- an Nginx container with FastCGI Caching, [Pagespeed](https://www.modpagespeed.com/) extensions and [NAXSI firewall](https://github.com/nbs-system/naxsi)
- a Google Cloud SQL proxy container to a on a Cloud SQL database server

## Prerequisites
* You need a Kubernetes cluster on Google Compute Engine. Follow the [official Kubernetes guide](https://cloud.google.com/kubernetes-engine/docs/quickstart "Running Kubernetes on Google Compute Engine").
* You need a Google Cloud SQL Database Server. Follow the [official Google Cloud SQL guide](https://cloud.google.com/sql/docs/mysql/quickstart "Running Google Cloud SQL").
* You should be comfortable with basic SQL statements, i.e. creating and managing DBs, users, grants.
* You also need a domain and access to its DNS settings. These instructions use the generic domain names mysite1.com and mysite2.com.
* Upon deploying WordPress you should install:
    * [Redis Object Cache](https://wordpress.org/plugins/redis-cache/ "Redis Object Cache plugin for WordPress") plugin to connect your site to the Redis `Deployment`
    * A cache-clearing plugin such as [NGINX Cache](https://wordpress.org/plugins/nginx-cache/) if you want to make sure changes appear on your website promptly. There are also other plugins such as [NGINX Helper](https://wordpress.org/plugins/nginx-helper/) but this requires an additional NGINX module and we have not successfully tested this plugin.

## Installation

1. Install Kubernetes WordPress Charts
- a. Download/clone [Kubernetes WordPress](https://github.com/stcox/k8s-wordpress.git) project
- b. 'cd' to project root.

2. [Create a Google Cloud SQL Instance](https://cloud.google.com/sql/docs/mysql/create-instance).

3. [Create a Cloud SQL Client Service Account and save credentials.json]()

4. Install Helm & Tiller
```bash
$ curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get | bash

$ kubectl create -f tiller-rbac-config.yaml
serviceaccount "tiller" created
clusterrolebinding "tiller" created

$ helm init --service-account tiller
$HELM_HOME has been configured at $HOME/.helm.
Tiller (the Helm server-side component) has been installed into your Kubernetes Cluster.
```

5. Install core services: Nginx-Ingress, Kube-Lego and Redis
```bash
$ helm install nginx-ingress
$ helm install kube-lego
$ helm install redis
```

6. In the project root folder, create and change to a folder called 'wp-sites'. This folder will list site folders you've created and is included in .gitignore.

```bash
$ mkdir wp-sites
$ cd wp-sites
/wp-sites $
```

## Usage

### Adding a website

1. Create a persistent disk for `mysite1-com` files. Be sure to prefix the namespace with `wp-`.
```bash
$ gcloud compute disks create --size=5GB --zone=<**ZONE**> wp-**mysite-com**
# find your **ZONE** at https://console.cloud.google.com/compute/instanceGroups/list
```

2. Create two files called `.dbuser` and `.dbpw` and put them in the site folder `/wp-sites/mysite1-com`. Put



2. Add a namespace `mysite1-com`, with site domain `mysite1.com`:
  a. Create a `mysite1-com folder`, then cd to it.
  b. Make a copy of the default wordpress/values.yaml file
```bash
/wp-sites $ mkdir mysite1-com && cd mysite1-com
/wp-sites/mysite1-com $ cp ../../wordpress/values.yaml values.yaml
```

3. With your favorite editor, edit the `/wp-sites/mysite1-com/values.yaml` file and change the `name` value to `mysite1-com` and the `domain` value to `mysite1.com`, and save your changes.

5. Install WordPress helm chart with `mysite1-com` values.
```bash
/wp-sites/mysite1-com $ helm install -f values.yaml ../../wordpress
```

## Acknowledgements
This project was inspired by [daxio/k8s-lemp](https://github.com/daxio/k8s-lemp) and builds on it with the various other official Docker images and Kubernetes applications mentioned previously.
