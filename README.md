# mlchain-kubernetes

Deploy [Mlchain](https://mlchain.khulnasoft.com/) on Kubernetes

> Feel free to raise issues or email me if you need support 😊
[Email](mailto:info@khulnasoft.com)

> Star 🌟 if this repo help you ~~

## Development Plan

### Add ssrf proxy component

Integrated ssrf proxy component into `mlchain-deployment.yaml` and `mlchain-mirror-deployment.yaml`. You can get files in `mlchain/middleware`.

### Other vector database

**Welcome PR!**
I have a development plan for this and will start in October 2024.

You can get files in `mlchain/database`.

I create a new branch for HA database setup which is `feature/mlchain-database-HA-setup`, and a folder `database-ha` under folder `mlchain`. Feel free to add files if you want to contribute to HA database!


## How to use

### Clone the repository

```shell

git clone https://github.com/mlchain/mlchain-kubernetes.git

kubectl apply -f mlchain-kubernetes.yaml

```

### Apply Directly

```shell

kubectl apply -f https://raw.githubusercontent.com/mlchain/mlchain-kubernetes/main/mlchain-deployment.yaml

```

If cluster is not able to connect dockerhub directly(for most users in China), apply deployment with mirror registry preset below.

```shell

kubectl apply -f https://cdn.jsdelivr.net/gh/mlchain/mlchain-kubernetes@main/mlchain-mirror-deployment.yaml

```

After Deployed, you can visit the mlchain web site via nodeport at `http://$(PUBLIC_IP):30000`, the **default init password** is `password`, or you can deploy a ingress to your cluster.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: mlchain-ingress
  namespace: mlchain
spec:
  ingressClassName: "traefik"
  rules:
    - host: mlchain.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: mlchain-nginx
                port:
                  number: 80
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: mlchain-nginx
                port:
                  number: 80
          - path: /console/api
            pathType: Prefix
            backend:
              service:
                name: mlchain-nginx
                port:
                  number: 80
          - path: /v1
            pathType: Prefix
            backend:
              service:
                name: mlchain-nginx
                port:
                  number: 80
          - path: /files
            pathType: Prefix
            backend:
              service:
                name: mlchain-nginx
                port:
                  number: 80
  tls:
    - secretName: mlchain-tls
```

If you want to expose mlchain api, uninstall nginx component and deploy a ingress below, if you use nginx ingress controller, change this yaml file.

```yaml
# Traefik Ingress Route without nginx reverse proxy
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: mlchain-ingressroute
  namespace: mlchain
spec:
  entryPoints:
    - web
    - websecure
  routes:
    - kind: Rule
      # console web url
      match: Host(`mlchain.example.com`) && PathPrefix(`/`)
      middlewares:
        - name: ingress-cors
      services:
        - name: mlchain-web
          port: 3000
    - kind: Rule
      # app web url
      match: Host(`mlchainapp.example.com`) && PathPrefix(`/`)
      middlewares:
        - name: ingress-cors
      services:
        - name: mlchain-web
          port: 3000
    - kind: Rule
      # service api url
      match: Host(`mlchainapi.example.com`) && PathPrefix(`/`)
      middlewares:
        - name: ingress-cors
      services:
        - name: mlchain-api
          port: 5001
    - kind: Rule
      # console api url
      match: Host(`consoleapi.example.com`) && PathPrefix(`/`)
      middlewares:
        - name: ingress-cors
      services:
        - name: mlchain-api
          port: 5001

    - kind: Rule
      # app api url
      match: Host(`appapi.example.com`) && PathPrefix(`/`)
      middlewares:
        - name: ingress-cors
      services:
        - name: mlchain-api
          port: 5001
  tls:
    secretName: mlchain-tls
# Traefik Middleware for Ingress
---
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: ingress-cors
  namespace: mlchain
spec:
  headers:
    accessControlAllowCredentials: true
    accessControlAllowMethods:
      - "GET"
      - "OPTIONS"
      - "PUT"
      - "POST"
      - "DELETE"
      - "PATCH"
    accessControlAllowHeaders:
      # - "*"
      - "Content-Type"
      - "authorization"
      - "x-app-code"
    accessControlAllowOriginList:
      # - "*"
      - "https://consoleapi.example.com"
      - "https://mlchain.example.com"
      - "https://mlchainapi.example.com"
      - "https://mlchainapp.example.com"
      - "https://appapi.example.com"
    accessControlMaxAge: 100
    addVaryHeader: true
```
## Star History

[![Star History Chart](https://api.star-history.com/svg?repos=mlchain/mlchain-kubernetes&type=Date)](https://star-history.com/#mlchain/mlchain-kubernetes&Date)
