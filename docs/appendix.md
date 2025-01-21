---
layout: default
title: APPENDIX - Important commands
---

## Minikube

Start / Stop:

```
minikube start
minikube stop
```

To create a tunnel and access a loadbancer, in a new terminal session. When requested, authenticate. **Keep this session open and active!**

```
minikube tunnel
```



## Bookinfo App

Access the Bookinfo application:

```
export INGRESS_HOST=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].port}')
echo http://$INGRESS_HOST:$INGRESS_PORT/productpage
```

Generate some load on Bookinfo instance. In a new session enter the following commands:

```
export INGRESS_HOST=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].port}')
watch curl http://$INGRESS_HOST:$INGRESS_PORT/productpage
```

This will access Bookinfo every 2 seconds until terminated. Keep this running during this exercise.

## Kiali

```
bin/istioctl dashboard kiali
```

Has to executed from the istio directory, opens Kiali in the default browser.