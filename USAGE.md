# Using GKE WordPress

## Pre-requisites
* **Google Kubernetes Engine cluster**. Follow the Google Kubernetes Engine [Creating a Cluster](https://cloud.google.com/kubernetes-engine/docs/how-to/creating-a-cluster "Creating a Cluster") guide.
* **Google Cloud SQL instance**. Follow the Google Cloud SQL [Creating instances](https://cloud.google.com/sql/docs/mysql/create-instance "Create Google Cloud SQL instance") guide.
* **Google Cloud SQL credentials** saved to your locally as `credentials.json`. You'll need them later. See [Connecting Cloud SQL to Kubernetes Engine](https://cloud.google.com/sql/docs/mysql/connect-kubernetes-engine).
* **A domain and access to its DNS settings**. These instructions use the generic domain name `mysite.com` as an example site domain. You should replace it with your own domain name.

## Installation
1. Install **GKE WordPress** project locally
```bash
$ mkdir -p gke-wp{wp-sites} && cd gke-wp
$ git clone https://github.com/stcox/gke-wordpress.git && cd gke-wordpress
```

2. Install **Helm & Tiller**
```bash
$ curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get | bash
$ kubectl create -f tiller-rbac-config.yaml
$ helm init --service-account tiller
```

3. Install core cluster services: **Nginx-Ingress, Kube-Lego and Redis**. Replace the kube-lego `legoEmail=` value with your own email address.
```bash
$ helm install nginx-ingress
$ helm install kube-lego --set legoEmail=myemail@mysite.com
$ helm install redis && cd ..
```

## Usage
### Adding sites
The example site uses `mysite-com`, for the site's namespace, and `mysite.com` for the domain. **All WordPress site namespaces are automatically prefixed with `wp-`** to make them easier to find; consequently, the example namespace will appear as **`wp-mysite-com`** in k8s.

The example domain name is, `mysite.com` must be substituted with your own domain.

Kube-lego provides free LetsEncrypt SSL certificates for any domains you control. LetsEncrypt is enabled by default, but can be disabled in the sites `values.yaml` file.

1. **Create an A record** for your domain, `mysite.com` at domain registrar (Godaddy, et al.), and point it to your Ingress IP address. [Get your cluster's Ingress IP Address](http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/#!/service?namespace=nginx-ingress)

2. **Configure site** in `values.yaml`.
```bash
$ cp ./gke-wordpress/wordpress/values.yaml ./wp-sites/mysite-com.yaml
$ nano ./wp-sites/mysite-com.yaml
```

3. **Deploy site** to cluster.
```bash
$ helm install -f ./wp-sites/mysite-com.yaml ./gke-wordpress/wordpress
```

## Post-requisites
* Upon deploying a site, edit the site and:
  * Install [**Redis Object Cache plugin**](https://wordpress.org/plugins/redis-cache/ "Redis Object Cache plugin for WordPress"), and select the **Connect** button to connect to Redis, and
  * Install [**NGINX Cache plugin**](https://wordpress.org/plugins/nginx-cache/) and set **Cache Zone Path** to `/var/run/nginx-cache`, and set Purge Cache, to ensure changes appear on your website promptly.


## Acknowledgements
This project was inspired by [daxio/k8s-lemp](https://github.com/daxio/k8s-lemp) and builds on it with the various other official Docker images and Kubernetes applications mentioned previously.

## Donate
[![Donate](https://img.shields.io/badge/Donate-PayPal-green.svg)](https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=FNLE7XYVKHSS2)
