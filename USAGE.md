# WordPress-Nginx-CloudSQL Kubernetes Helm Charts
Host multiple WordPress sites on Compute Engine and Google Cloud SQL.
Each site has its own Kubernetes Namespace with one or more Pods containing:
  - a WordPress/PHP-FPM container with Redis extensions
  - an Nginx container with FastCGI Caching, [Pagespeed](https://www.modpagespeed.com/) extensions and [NAXSI firewall](https://github.com/nbs-system/naxsi)
- a Google Cloud SQL proxy container that connects to a Cloud SQL database server

## Prerequisites

* You need a Kubernetes cluster on Google Compute Engine. Follow the [official Kubernetes guide](https://cloud.google.com/kubernetes-engine/docs/how-to/creating-a-container-cluster "Creating a Container Cluster").
* You need a Google Cloud SQL Database Server. Follow the [official Google Cloud SQL guide](https://cloud.google.com/sql/docs/mysql/create-instance "Create Google Cloud SQL instance").
* You need to save your Google Cloud SQL database credentials to your hard drive somewhere as `credentials.json`. You'll copy it to a different destination later. You should be comfortable with basic SQL statements, i.e. creating and managing DBs, users, grants.
* You also need a domain and access to its DNS settings. These instructions uses the generic domain names mysite1.com as an example sites.
* Upon deploying WordPress you should install:
  * [Redis Object Cache](https://wordpress.org/plugins/redis-cache/ "Redis Object Cache plugin for WordPress") plugin to connect your site to the Redis `Deployment`
  * A cache-clearing plugin such as [NGINX Cache](https://wordpress.org/plugins/nginx-cache/) if you want to make sure changes appear on your website promptly. There are also other plugins such as [NGINX Helper](https://wordpress.org/plugins/nginx-helper/) but this requires an additional NGINX module and we have not successfully tested this plugin.

## Installation

1. Install Helm & Tiller

```bash
$ curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get | bash

$ kubectl create -f tiller-rbac-config.yaml
serviceaccount "tiller" created
clusterrolebinding "tiller" created

$ helm init --service-account tiller
$HELM_HOME has been configured at $HOME/.helm.
Tiller (the Helm server-side component) has been installed into your Kubernetes Cluster.
```

1. Install Kubernetes WordPress Helm Charts to local computer
  - 1a. Download/clone [Kubernetes WordPress](https://github.com/stcox/k8s-wordpress.git) project
  - 1b. 'cd' to project root.

3. Install core services: Nginx-Ingress, Kube-Lego and Redis
```bash
$ helm install nginx-ingress
$ helm install kube-lego
$ helm install redis
```

4. In the project root folder, create and change to a folder called 'wp-sites'. This folder will list site folders you've created and is included in .gitignore.

```bash
$ mkdir wp-sites
$ cd wp-sites
/wp-sites $
```
5. Copy Cloud SQL credentials.json file to /wp-sites

## Usage

### Adding a website

This example uses mysite1-com for a namespace and mysite1.com for a domain. This default domain will only work with HTTP. HTTPS/SSL is free with any domain you own. You can change the namespace and domain to reflect a domain you've registered. To use your own domain:

1. At your domain name provider (Godaddy, Bluehost, etc.), create an A record for your domain, `mysite1.com` in this example, and point it to your ingress. [Click here to get your cluster ingress ip address](http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/#!/service?namespace=nginx-ingress)

2. Create a **persistent disk** for `mysite1-com` files. Be sure to add the namespace prefix: **wp-**
```bash
$ gcloud compute disks create --size=5GB --zone=<**ZONE**> wp-mysite-com
# find your <**ZONE**> at https://console.cloud.google.com/compute/instanceGroups/list
```

3. Create login files:
  a. Create a file named wp-sites/mysite1-com/.dbuser and enter a new database **username**.
	b. Create a file named wp-sites/mysite1-com/.dbpw and enter a new database **password**.

4. Configure namespace `mysite1-com`:
  a. Create a `mysite1-com` folder, then `cd` to it.
  b. Put a copy of the default wordpress/values.yaml file in /wp-sites/mysite1-com folder.
```bash
/wp-sites $ mkdir mysite1-com && cd mysite1-com
/wp-sites/mysite1-com $ cp ../../wordpress/values.yaml values.yaml
```

5. With your favorite editor, edit the `/wp-sites/mysite1-com/values.yaml` file and change the `name` value to `mysite1-com` and the `domain` value to `mysite1.com`, and save your changes.

6. Install WordPress helm chart with `mysite1-com` values.
```bash
/wp-sites/mysite1-com $ helm install -f values.yaml ../../wordpress
```

## Acknowledgements
This project was inspired by [daxio/k8s-lemp](https://github.com/daxio/k8s-lemp) and builds on it with the various other official Docker images and Kubernetes applications mentioned previously.
