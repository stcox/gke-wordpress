# Wordpress on Google Kubernetes Engine

## Pre-requisites
* **Git**
* **Google Kubernetes Engine cluster**. Follow the Google Kubernetes Engine [Creating a Cluster](https://cloud.google.com/kubernetes-engine/docs/how-to/creating-a-cluster "Creating a Cluster") guide. When creating a cluster, make sure you select a Kubernetes version 1.9 or greater.
* **Google Cloud SQL instance**. Follow the Google Cloud SQL [Creating instances](https://cloud.google.com/sql/docs/mysql/create-instance "Create Google Cloud SQL instance") guide.
* **Google Cloud SQL credentials** saved to your locally as `credentials.json`. You'll need them later. See [Connecting Cloud SQL to Kubernetes Engine](https://cloud.google.com/sql/docs/mysql/connect-kubernetes-engine).
* [Optional] **A domain and access to its DNS settings**. These instructions use the generic domain name `mysite.com` as an example site domain. You should replace it with your own domain name.

# Deploy Wordpress Easy Way
1. Using wordpress google cloud marketplace, configure & deploy. https://console.cloud.google.com/marketplace/details/google/wordpress
2. Connect to your GKE (Google Kubernetes Engine)
```bash
$ gcloud container clusters get-credentials {YOURGKECLUSTERNAME} --zone {YOURGKECLUSTERZONE} --project {YOURPROJECTNAME}
```
3. Expose your wordpress service from your google cloud shell
```bash
$ export APP_INSTANCE_NAME={YOURWORDPRESSSERVICENAME} # From Step 1
$ export NAMESPACE={YOURWORDPRESSNAMESPACE} # From Step 1
$ kubectl patch svc "$APP_INSTANCE_NAME-wordpress-svc" \
  --namespace "$NAMESPACE" \
  --patch '{"spec": {"type": "LoadBalancer"}}'
```
4. Get your wordpress site address
```bash
$ SERVICE_IP=$(kubectl get svc $APP_INSTANCE_NAME \
  --namespace $NAMESPACE \
  --output jsonpath='{.status.loadBalancer.ingress[0].ip}')
$ echo "http://${SERVICE_IP}"
```

5. Scale up or down your wordpress application
```bash
$ kubectl scale deployment/{YOURWORDPRESSPOD_NAME} --replicas=N #Example 3 for scaling up to 3 pods, 1 for scaling down to 1 pods
```

### Notes
*If you want to simplify above script, just create bash script ie: `publish.sh` and copy all command from step 2-4 into it.


# Deploy Wordpress Hard Way using Helm

## Installation
The example site uses `mysite-com`, for the site's namespace, and `mysite.com` for the domain. **All WordPress site namespaces are automatically prefixed with `wp-`** to make them easier to find; consequently, the example namespace will appear as **`wp-mysite-com`** in k8s.

The example domain name is, `mysite.com` must be substituted with your own domain.

Kube-lego provides free LetsEncrypt SSL certificates for any domains you control. LetsEncrypt is enabled by default, but can be disabled in the sites `values.yaml` file.

1. **Create an A record** for your domain, `mysite.com` at domain registrar (Godaddy, et al.), and point it to your Ingress IP address. [Get your cluster's Ingress IP Address](https://console.cloud.google.com/kubernetes/discovery). Or just edit in your /etc/host for testing purpose.

2. Install **GKE WordPress** project locally
```bash
$ git clone https://github.com/ekarisky/gke-wordpress.git && cd gke-wordpress
```

3. **Configure site** in `values.yaml`.
```bash
$ mkdir wp-sites
$ cp gke-wordpress/wordpress/values.yaml wp-sites/mysite-com.yaml
$ nano wp-sites/mysite-com.yaml
```

4. Install **Helm & Tiller**
```bash
$ curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get | bash
$ kubectl create -f tiller-rbac-config.yaml
$ helm init --service-account tiller
```

5. Install core cluster services: **Nginx-Ingress, Kube-Lego, Redis, and Dynamically Provisioned Storage Classes**.
```bash
$ helm install nginx-ingress
$ helm install kube-lego --set legoEmail=MYEMAIL@MYSITE.COM
$ helm install redis
$ kubectl create -f storageclass.yaml
```

6. **Deploy site** to cluster.
```bash
$ helm install -f wp-sites/mysite-com.yaml wordpress
```

7. Scale up or down your wordpress application
```bash
$ kubectl scale deployment/{YOURWORDPRESSPOD_NAME} --replicas=N #Example 3 for scaling up to 3 pods, 1 for scaling down to 1 pods
```

### Notes
*If you want to simplify above script, just create bash script ie: `publish.sh` and copy all command from step 2-6 into it.

## Post-requisites
* Upon deploying a site, edit the site and:
  * Install [**Redis Object Cache plugin**](https://wordpress.org/plugins/redis-cache/ "Redis Object Cache plugin for WordPress"), and select the **Connect** button to connect to Redis, and
  * Install [**NGINX Cache plugin**](https://wordpress.org/plugins/nginx-cache/) and set **Cache Zone Path** to `/var/run/nginx-cache`, and set Purge Cache, to ensure changes appear on your website promptly.


## Acknowledgements
This project was forked from [stcox/gke-wordpress](https://github.com/stcox/gke-wordpress), and modified accordingly
This project was inspired by [daxio/k8s-lemp](https://github.com/daxio/k8s-lemp) and builds on it with the various other official Docker images and Kubernetes applications mentioned previously.
