---
title:  "Setup local docker repository!"
date:   2020-05-14 14:32:16 +0900
categories: docker
author: Prabhat Kumar
---
# Setup local docker registry inside k8s cluseter

## Install helm client (macOS)
```
brew install kubernetes-helm
```
## Install tiller
```
helm init
helm repo update
```
## Deploy the NGINX Ingress Controller
```
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm install my-nginx-controller  ingress-nginx/ingress-nginx
POD_NAME=$(kubectl get pods -l app.kubernetes.io/name=ingress-nginx -o jsonpath='{.items[0].metadata.name}')
kubectl exec -it $POD_NAME -- /nginx-ingress-controller --version
kubectl get svc
```
*Make sure nodeport for nginx-ingress-controller is allowed for firewall* or apache forward proxy configured, to access node port from internet.
## Assign a DNS name
The external IP that is allocated to the ingress-controller is the IP to which all incoming traffic should be routed. To enable this, add it to a DNS zone you control, for example as example.your-domain.com.

This quick-start assumes you know how to assign a DNS entry to an IP address and will do so.

## Docker registry
```
kubectl apply -f https://github.com/prabhatmaurya/plabs-sre-admin/raw/master/k8s-labs/01_deploy_local_registry/deployment.yaml --validate=false
```
### Add ingress for registry
```
wget https://raw.githubusercontent.com/prabhatmaurya/plabs-sre-admin/master/k8s-labs/01_deploy_local_registry/ingress-registry-v1.yaml
## replace DNS name in above file
kubectl apply -f ingress-registry-v1.yaml
##verify ingress
kubectl -n docker-registry get ingress
## verify registry access
curl -kivL  https://<domain>:<ingress nodeport>/v2/_catalog
```

## Deploy Cert Manager
```
kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v0.15.0/cert-manager.yaml
## Verify
kubectl get pods --namespace cert-manager
```
## Configure Let’s Encrypt Issuer
### Install stage issuer
```
wget https://raw.githubusercontent.com/prabhatmaurya/plabs-sre-admin/master/k8s-labs/01_deploy_local_registry/issuer-staging.yaml
## replace email with your email
kubectl apply -f issuer-staging.yaml
kubectl -n docker-registry describe issuer letsencrypt-staging
```
### Install prod issuer
```
wget https://raw.githubusercontent.com/prabhatmaurya/plabs-sre-admin/master/k8s-labs/01_deploy_local_registry/issuer-prod.yaml
## replace email with your email
kubectl apply -f issuer-prod.yaml
kubectl -n docker-registry describe issuer letsencrypt-prod
```

### Deploy a TLS Ingress Resource with stagging
```
wget https://raw.githubusercontent.com/prabhatmaurya/plabs-sre-admin/master/k8s-labs/01_deploy_local_registry/ingress-registry-v2.yaml
## replace DNS name in above file
kubectl apply -f ingress-registry-v2.yaml
##verify ingress
kubectl -n docker-registry get ingress
## get certificate
kubectl -n docker-registry describe certificate
### if output is "Issued		Certificate issued successfully"  then setup is working and skip to next section else follow next step for debugging

## get certificate request status
kubectl -n docker-registry describe certificateRequest <request id>

## Get cert request order status
kubectl -n docker-registry describe order <order id>
```
### Deploy a TLS Ingress Resource with prod
```
wget https://raw.githubusercontent.com/prabhatmaurya/plabs-sre-admin/master/k8s-labs/01_deploy_local_registry/ingress-registry-v3.yaml
## replace DNS name in above file
kubectl apply -f ingress-registry-v3.yaml
##verify ingress
kubectl -n docker-registry get ingress

## verify new prod cert
kubectl -n docker-registry describe certificate

## test docker registry
curl -kivL  https://<domain>:<ingress nodeport>/v2/_catalog
```
# References:-
* https://kubernetes.github.io/ingress-nginx/examples/docker-registry/
* https://cert-manager.io/docs/tutorials/acme/ingress/
* https://cert-manager.io/docs/installation/kubernetes/
