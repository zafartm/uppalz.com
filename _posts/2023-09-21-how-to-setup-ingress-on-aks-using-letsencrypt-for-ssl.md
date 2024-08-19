---
layout: post
title:  "How to setup ingress on AKS (Azure k8s service) using letsencrypt certs for SSL"
---
# How to setup ingress on AKS (Azure k8s service) using letsencrypt certs for SSL

### Install cert-manager
```shell
helm repo add jetstack https://charts.jetstack.io
helm repo update
helm upgrade cert-manager jetstack/cert-manager --install --create-namespace --wait --namespace cert-manager --set installCRDs=true
```
Above commands are taken from https://cert-manager.io/docs/installation/helm/

### Install ingress-nginx controller and load balancer
DO NOT FOLLOW official docs at https://learn.microsoft.com/en-us/azure/aks/ingress-basic?tabs=azure-cli. It doesn't work and wastes a lot of time.

Instead. use the following command which I took from https://kubernetes.github.io/ingress-nginx/deploy/#azure

```shell
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.2/deploy/static/provider/cloud/deploy.yaml
```

### Get the external IP address of the load balancer
```shell
kubectl get services --namespace ingress-nginx -o wide
```

### Create a ClusterIssuer using this yaml
```yaml
---
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt
spec:
  acme:
    email: "a valid email address of your own"
    server: https://acme-v02.api.letsencrypt.org/directory
    ## Use this staging server during testing and exploration to avoid rate limits
    #server: https://acme-staging-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      name: letsencrypt-account-key
    solvers:
    - http01:
        ingress:
          ingressClassName: nginx
---
```

### Add DNS record
Add an "A" record in the DNS server (cloudflare, godaddy etc) for the above IP and your desired domain name, e.g my-own-domain.com

### Deploy ingress for my-own-domain.com
```yaml
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt
  name: my-own-ingress
  namespace: default
spec:
  ingressClassName: nginx
  defaultBackend:
    service:
      name: my-frontend-service-on-cluster-ip
      port:
        number: 8080
  rules:
  - host: my-own-domain.com
    http:
      paths:
      - backend:
          service:
            name: my-backend-service-on-cluster-ip
            port:
              number: 8080
        path: /api/
        pathType: Prefix
  tls:
  - hosts:
    - my-own-domain.com
    secretName: my-own-domain-tls-cert
---
```