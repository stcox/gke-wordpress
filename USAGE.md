# Using Kubernetes WordPress
## Prerequisites
* You need a Kubernetes cluster on Google Compute Engine. Follow the [official Kubernetes guide](https://cloud.google.com/kubernetes-engine/docs/how-to/creating-a-container-cluster "Creating a Container Cluster").
* You need a Google Cloud SQL Database Server. Follow the [official Google Cloud SQL guide](https://cloud.google.com/sql/docs/mysql/create-instance "Create Google Cloud SQL instance").
* You need your Google Cloud SQL database credentials saved to your hard drive somewhere as `credentials.json`. You'll copy it to a different destination later. You should be comfortable with basic SQL statements, i.e. creating and managing DBs, users, grants.
* You need a domain and access to its DNS settings. These instructions use the generic domain name `mysite.com` as an example site.
* Upon deploying WordPress you should install:
  * [Redis Object Cache](https://wordpress.org/plugins/redis-cache/ "Redis Object Cache plugin for WordPress") plugin to connect your site to the Redis `Deployment`
  * The cache-clearing plugin [NGINX Cache](https://wordpress.org/plugins/nginx-cache/) if you want to make sure changes appear on your website promptly.
* You need Helm & Tiller installed. Follow the guide to [Install Helm & Tiller](https://docs.helm.sh/using_helm/#installing-helm), or go to the project directory and run:
```bash
$ curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get | bash

$ kubectl create -f tiller-rbac-config.yaml
serviceaccount "tiller" created
clusterrolebinding "tiller" created

$ helm init --service-account tiller
$HELM_HOME has been configured at $HOME/.helm.
Tiller (the Helm server-side component) has been installed into your Kubernetes Cluster.
```

## Installation
1. Install Kubernetes WordPress Helm Charts to local computer
```bash
$ mkdir -p k8s-wp{wp-secrets,wp-sites} && cd k8s-wp

k8s-wp/ $ git clone https://github.com/stcox/k8s-wordpress.git

k8s-wp/ $ cd k8s-wordpress
```

2. Install core services: Nginx-Ingress, Kube-Lego and Redis
From project root folder, run:
```bash
$ helm install nginx-ingress && helm install kube-lego && helm install redis
```

3. From project root, create and change to folder `wp-sites/`. This is where you'll keep your site secrets and configuration values.yaml, and it's included in .gitignore.
```bash
$ mkdir wp-sites && cd wp-sites
wp-sites/ $
```

4. Copy Cloud SQL credentials.json file to /wp-sites

## Usage
### Adding a website
This example uses mysite-com for a namespace and mysite.com for a domain. This default domain will only work with HTTP. HTTPS/SSL/HTTP2 is included with any domain you actually own and is enabled by setting `tls: true` in the site's `values.yaml` file. You can change the namespace and domain to reflect a domain you've registered.

1. Configure a namespace `mysite-com`:
  - a. Create a `wp-sites/mysite-com` folder and make it present.
  - b. Copy the default `k8s-wordpress/wordpress/values.yaml` file into `/wp-sites/mysite-com folder/`.
```bash
/wp-sites $ mkdir mysite-com && cd mysite-com
/wp-sites/mysite-com $ cp ../../wordpress/values.yaml values.yaml
```



1. At your domain name provider (Godaddy, Bluehost, etc.), create an A record for your domain, `mysite.com` in this example, and point it to your ingress ip address. [Click here to get your cluster's ingress ip address](http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/#!/service?namespace=nginx-ingress)

2. Create a **persistent disk** for `mysite-com` files. Be sure to add the namespace prefix: **wp-**. The size should match the size you specified in the
```bash
$ gcloud compute disks create --size=5GB --zone=<**ZONE**> wp-mysite-com
# find your <**ZONE**> at https://console.cloud.google.com/compute/instanceGroups/list
```

3. Create database secrets:
  - a. Create a file named wp-sites/mysite-com/.dbuser and enter a new database **username**.
	- b. Create a file named wp-sites/mysite-com/.dbpw and enter a new database **password**.


5. With your favorite editor, edit the `/wp-sites/mysite-com/values.yaml` file and change the `name` value to `mysite-com` and the `domain` value to `mysite.com`, and save your changes.

6. Install WordPress helm chart with `mysite-com` values.
```bash
/wp-sites/mysite-com $ helm install -f values.yaml ../../wordpress
```

## Acknowledgements
This project was inspired by [daxio/k8s-lemp](https://github.com/daxio/k8s-lemp) and builds on it with the various other official Docker images and Kubernetes applications mentioned previously.
