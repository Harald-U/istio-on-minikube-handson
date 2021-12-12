---
layout: default
title: APPENDIX - Important commands
---

## Minikube

Start / Stop:

```
$ minikube start
$ minikube stop
```

Create tunnel and access loadbancer, in a new terminal session. When requested, authenticate. **Keep this session open and active!**

```
$ minikube tunnel
```



## Bookinfo App

Access the Bookinfo application:

```
$ export INGRESS_HOST=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
$ export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].port}')
$ echo http://$INGRESS_HOST:$INGRESS_PORT/productpage
http://10.107.101.80:80/productpage
```

Generate some load on Bookinfo instance. In a new session enter the following commands:

```
$ export INGRESS_HOST=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
$ export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].port}')
$ watch curl http://$INGRESS_HOST:$INGRESS_PORT/productpage
```

This will access Bookinfo every 2 seconds until terminated. Keep this running during this exercise.

## Kiali

```
$ kubectl port-forward service/kiali 20001:20001 -n istio-system
```

In a browser open the Kiali dashboard at http://localhost:20001/