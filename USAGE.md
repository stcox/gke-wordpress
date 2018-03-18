# Using Kubernetes WordPress
## Prerequisites
* You need a Kubernetes cluster on Google Compute Engine. Follow the [official Kubernetes guide](https://cloud.google.com/kubernetes-engine/docs/how-to/creating-a-container-cluster "Creating a Container Cluster").
* You need a Google Cloud SQL Database Server. Follow the [Creating a Google Cloud SQL guide](https://cloud.google.com/sql/docs/mysql/create-instance "Create Google Cloud SQL instance") and [Connecting Cloud SQL to Kubernetes Engine](https://cloud.google.com/sql/docs/mysql/connect-kubernetes-engine) guide.
* You need your Google Cloud SQL database credentials saved to your hard drive somewhere as `credentials.json`. You'll need it later. You should be comfortable with basic SQL statements, i.e. creating and managing DBs, users, grants.
* You need a domain and access to its DNS settings. These instructions use the generic domain name `mysite.com` as an example site.
* Upon deploying WordPress you should install:
  * [Redis Object Cache](https://wordpress.org/plugins/redis-cache/ "Redis Object Cache plugin for WordPress") plugin to connect your site to the Redis `Deployment`
  * The cache-clearing plugin [NGINX Cache](https://wordpress.org/plugins/nginx-cache/) if you want to make sure changes appear on your website promptly.

## Installation
1. Install Kubernetes WordPress to local computer
```bash
$ mkdir -p k8s-wp{wp-secrets,wp-sites} && cd k8s-wp
k8s-wp/ $ git clone https://github.com/stcox/k8s-wordpress.git
```

2. Install Helm & Tiller
```bash
k8s-wp/ $ cd k8s-wordpress
k8s-wp/k8s-wordpress/ $ curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get | bash
k8s-wp/k8s-wordpress/ $ kubectl create -f tiller-rbac-config.yaml
k8s-wp/k8s-wordpress/ $ helm init --service-account tiller
```

3. Install core services: Nginx-Ingress, Kube-Lego and Redis
```bash
k8s-wp/k8s-wordpress/ $ helm install nginx-ingress && helm install kube-lego --set legoEmail=`myemail@mysite.com` && helm install redis
```

## Usage
### Adding websites
The example uses `mysite-com`, for the site's namespace, and `mysite.com` for the domain. All WordPress namespaces are prefixed with `wp-` to create the site's namespace, so they are easier to find; consequently, the example namespace will appear as `wp-mysite-com` in kubernetes.

The example domain, `mysite.com` only works with HTTP via a local hosts file. You should substitute your own domain.

Free LetsEncrypt TLS/SSL/HTTPS/HTTP2 certificates are available for any domains you control. LetsEncrypt is enabled by setting `tls: true` in the site's configuration file.

1. Configure your site, `mysite-com`. Name, domain, dbUser, dbPass, dbConn, and dbCred MUST be changed:
```bash
k8s-wp/k8s-wordpress/ $ cd ../wp-sites
k8s-wp/wp-sites/ $ cp ../k8s-wordpress/wordpress/values.yaml mysite-com.yaml
k8s-wp/wp-sites/ $ nano mysite-com.yaml
```

2. Create a **persistent disk** for `mysite-com`.
```bash
k8s-wp/wp-sites/ $ gcloud compute disks create --size=5GB --zone=<**ZONE**> mysite-com
# find your <**ZONE**> at https://console.cloud.google.com/compute/instanceGroups/list
```

3. At your domain name provider (Godaddy, Bluehost, etc.), create an A record for your domain, `mysite.com` in this example, and point it to your ingress ip address. [Click here for your cluster's ingress ip address](http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/#!/service?namespace=nginx-ingress)

4. Install WordPress helm chart with `mysite-com` values.
```bash
k8s-wp/wp-sites/ $ helm install -f mysite-com.yaml ../k8s-wordpress/wordpress
```

## Acknowledgements
This project was inspired by [daxio/k8s-lemp](https://github.com/daxio/k8s-lemp) and builds on it with the various other official Docker images and Kubernetes applications mentioned previously.

## Donate
[![Donate](https://img.shields.io/badge/Donate-PayPal-green.svg)](https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=FNLE7XYVKHSS2)
