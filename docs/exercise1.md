---
layout: default
title: 1. Setting up Minikube and Istio
---

### 1 Create a Kubernetes instance on Minikube

I am going to assume that you already have Minikube installed on your workstation. If this is not the case follow the instructions in the [Minikube documention](https://minikube.sigs.k8s.io/docs/start/).

To start a Kubernetes instance enter the following command in a shell:

```
minikube start --cpus 2 --memory 4096 --driver docker
```

This will start an instance with 2 virtual CPUs, 4 GB om RAM, using Docker (Desktop) as your virtualization platform.

### NOTE: bwLehrpool

<!-- >> There may be a "leftover" (and damaged) Minikube instance that was present when the VMware image for the Linux environment was built.  This may cause problems. Enter the following command before you start this workshop:

```
minikube delete
```

> Output will look something like this:

```
🔥  minikube" in docker wird gelöscht...
🔥  /home/student/.minikube/machines/minikube wird entfernt...
💀  Removed all traces of the "minikube" cluster.
```

> Please be aware that this command will delete any existing Minikube cluster! -->

bwLehrpool has sufficient RAM to increase memory for Minikube, you can use this command instead:

```
minikube start --cpus 2 --memory 6144 --driver docker
```

which will assign 6 GB of RAM.

**NOTE 2:** According to the [Istio documentation](https://istio.io/latest/docs/setup/platform-setup/minikube/) a Minikube instance with at least 4 virtual CPUs and 16 GB of RAM is required. I have tested this workshop with the smaller configuration and it works but of course will not win a price for high performance.

### 2 Install Istio

This workshop is based on Istio version 1.16.1  (which was released in December 2022).

Official instructions can be found [here](https://istio.io/latest/docs/setup/getting-started/).

1. Create a new directory in your home directory, e.g. 'vs' and change into it

    **Note: On bwLehrpool** change into the PERSISTENCE directory before continuing with the next step.

2. Download Istio 1.16.1:
   
    ```
	curl -L https://istio.io/downloadIstio | ISTIO_VERSION=1.16.1 TARGET_ARCH=x86_64 sh -
    ```

3. Change into the Istio directory
   
    ```
	cd istio-1.16.1
    ```

4. Install Istio:

    ```
	bin/istioctl install --set profile=demo -y
    ```

   Output: 

    ```
    ✔ Istio core installed                                                                                                  
    ✔ Istiod installed                                                                                                      
    ✔ Ingress gateways installed                                                                                            
    ✔ Egress gateways installed                                                                                             
    ✔ Installation complete  
    Making this installation the default for injection and validation.

    Thank you for installing Istio 1.12.
    ```

5. Verify the the Istio installation:

   Istio is installed into the *istio-system* namespace on Kubernetes. 

    ```
    kubectl get pod -n istio-system
    ```

    Output looks like this:

    ```
    NAME                                   READY   STATUS    RESTARTS   AGE
    istio-egressgateway-7f4864f59c-jz6f9   1/1     Running   0          4m47s
    istio-ingressgateway-55d9fb9f-592zs    1/1     Running   0          4m47s
    istiod-555d47cb65-ss54h                1/1     Running   0          5m12s
    ```

    The pod identifiers will be different but there should be 3 pods for egress gateway, ingress gateway, and istiod, all in status 'Running'.

    ```
    kubectl get svc -n istio-system
    ```

    Output looks like this:

    ``` 
    NAME                   TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                                                                      AGE
    istio-egressgateway    ClusterIP      10.100.115.226   <none>        80/TCP,443/TCP                                                               7m2s
    istio-ingressgateway   LoadBalancer   10.107.101.80    <pending>     15021:31820/TCP,80:31043/TCP,443:30723/TCP,31400:31291/TCP,15443:31719/TCP   7m2s
    istiod                 ClusterIP      10.96.232.106    <none>        15010/TCP,15012/TCP,443/TCP,15014/TCP                                        7m27s
    ```

    Output should show 3 services, again for egress gateway, ingress gateway, and istiod.

6. **Very important:** Enable automatic sidecar injection for default namespace

    ```
	kubectl label namespace default istio-injection=enabled	
    ```

   > Without this setting we will not use Istio although it is installed! 


### 3 Install Telemetry Addons

We will now install the telemetry or observability add-ons: 
* [Prometheus](https://istio.io/latest/docs/ops/integrations/prometheus/) (Monitoring)
* [Grafana](https://istio.io/latest/docs/ops/integrations/grafana/) (Monitoring)
* [Jaeger](https://istio.io/latest/docs/ops/integrations/jaeger/) (Distributed Tracing)
* [Kiali](https://istio.io/latest/docs/ops/integrations/kiali/) (Istio dashboard)

While still in the istio-1.12.0 directory, issue the following commands

```
kubectl apply -f samples/addons/prometheus.yaml
kubectl apply -f samples/addons/grafana.yaml
kubectl apply -f samples/addons/jaeger.yaml
kubectl apply -f samples/addons/kiali.yaml
```

Verify:

```
kubectl get pod -n istio-system
```

It will take a while for all the new pods to start, this is pushing the tiny cluster to its limits.

Output:

```
NAME                                   READY   STATUS    RESTARTS   AGE
grafana-6ccd56f4b6-2jnd7               1/1     Running   0          2m11s
istio-egressgateway-7f4864f59c-jz6f9   1/1     Running   0          17m
istio-ingressgateway-55d9fb9f-592zs    1/1     Running   0          17m
istiod-555d47cb65-ss54h                1/1     Running   0          17m
jaeger-5d44bc5c5d-r9mp5                1/1     Running   0          2m3s
kiali-79b86ff5bc-fpzd7                 1/1     Running   0          117s
prometheus-64fd8ccd65-2dgdc            2/2     Running   0          2m18s
```

And for the services:

```
kubectl get svc -n istio-system
```

Output:

```
NAME                   TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                                                                      AGE
grafana                ClusterIP      10.96.207.7      <none>        3000/TCP                                                                     2m43s
istio-egressgateway    ClusterIP      10.100.115.226   <none>        80/TCP,443/TCP                                                               17m
istio-ingressgateway   LoadBalancer   10.107.101.80    <pending>     15021:31820/TCP,80:31043/TCP,443:30723/TCP,31400:31291/TCP,15443:31719/TCP   17m
istiod                 ClusterIP      10.96.232.106    <none>        15010/TCP,15012/TCP,443/TCP,15014/TCP                                        18m
jaeger-collector       ClusterIP      10.107.97.241    <none>        14268/TCP,14250/TCP,9411/TCP                                                 2m34s
kiali                  ClusterIP      10.97.181.104    <none>        20001/TCP,9090/TCP                                                           2m28s
prometheus             ClusterIP      10.106.105.122   <none>        9090/TCP                                                                     2m49s
tracing                ClusterIP      10.98.100.68     <none>        80/TCP,16685/TCP                                                             2m34s
zipkin                 ClusterIP      10.103.150.84    <none>        9411/TCP                                                                     2m34s
```

Jaeger deployment creates 2 services: jaeger-collector and tracing. The tracing service will later provide the Jaeger UI.

---

## >> [Continue with Exercise 2](exercise2.md)