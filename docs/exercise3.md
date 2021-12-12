---
layout: default
title: 3. Observability
---

## Challenges with microservices

We all know that a microservice architecture is possibly the most perfect fit for cloud native applications and it increases the speed of delivery greatly. But envision you have many microservices that are delivered by multiple teams. How do you observe the overall platform and each of the services to find out exactly what is going on with each of the services? When something goes wrong, how do you know which service or which communication among the services is causing the problem?

## Istio telemetry

Istio integrates very well with a range of (open-source) telemetry and observability tools that can provide broad and granular insight into the health of all services. Istio’s role as a service mesh makes it the ideal data source for observability information, particularily in a microservices environment. 

As requests pass through multiple services, identifying performance bottlenecks becomes increasingly difficult using traditional debugging techniques. Distributed tracing using e.g. Jaeger provides a holistic view of requests transiting through multiple services, allowing for immediate identification of latency issues. With Istio, distributed tracing comes by default. This will expose latency, retry, and failure information for each hop in a request.

In Exercise 1, you have installed 4 telemetry or observability add-ons: 

* [Prometheus](https://istio.io/latest/docs/ops/integrations/prometheus/) (for Monitoring)
* [Grafana](https://istio.io/latest/docs/ops/integrations/grafana/) (for Monitoring)
* [Jaeger](https://istio.io/latest/docs/ops/integrations/jaeger/) (for Distributed Tracing)
* [Kiali](https://istio.io/latest/docs/ops/integrations/kiali/) (the Istio dashboard)

There is a whole section on [Observability](https://istio.io/latest/docs/tasks/observability/) in the Istio documentation.

## Accessing the Telemetry services

Look at the installed Telemetry services:

```
$ kubectl get svc -n istio-system
```

You can see that none of the Telemetry services is of type LoadBalancer or NodePort which means they are not accessible from the outside. In the following examples we will use Kubernetes [port-forwarding](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#port-forward) to compensate for this.

But first start to generate some load on your Bookinfo instance. In yet another new session enter the following commands:

```
$ export INGRESS_HOST=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
$ export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].port}')
$ watch curl http://$INGRESS_HOST:$INGRESS_PORT/productpage
```

Keep this running during this exercise.

### Jaeger

To start the Port Forwarding issue the following command in a seperate session (and keep it running):

```
$ kubectl port-forward service/tracing  8008:80 -n istio-system 
```

1. In a browser open the Jaeger UI at http://localhost:8008/.
2. As service, select 'istio-ingressgateway.istio-system'.
3. Click on 'Find Traces'.

    ![Jaeger UI](/images/jaeger1.png)

4. Click on one of the trace entries to see the details:

    ![Jaeger Details](/images/jaeger2.png)

    On the left side you can see how the request is passed through the different services: coming in via Istio Ingress, Productpage to Details, back to Productpage, Productpage to Reviews, Reviews to Ratings. An the graph shows how much time is being spent in each service.

When you are finished with Jaeger, terminate the Port Forwarding (Linux: Ctl+C, Mac: Cmd+.)

### Grafana

To start the Port Forwarding issue the following command in a seperate session (and keep it running):

```
$ kubectl port-forward service/grafana  3000:3000 -n istio-system 
```

1. In a browser open the Grafana UI at http://localhost:3000/
2. Click on the Looking Glas (search), click on the Istio folder, then select the Istio Performance Dashboard

    ![Grafana UI](/images/grafana1.png)

    This is general performance information collected from the service mesh:

    ![Grafana Details](/images/grafana2.png)

3. Look at the other Grafana Dashboards.    

When you are finished with Grafana, terminate the Port Forwarding (Linux: Ctl+C, Mac: Cmd+.)

### Prometheus

To start the Port Forwarding issue the following command in a seperate session (and keep it running):

```
$ kubectl port-forward service/prometheus 9090:9090 -n istio-system 
```

1. In a browser open the Prometheus UI at http://localhost:9090/
2. In the Query field start to type 'istio_requests_total'. When you start to type, Prometheus will generate a list of predefined metrics queries.
3. Select 'istio_requests_total', the click 'Execute' and select the 'Graph' tab.

    ![Prometheus UI](/images/prometheus1.png)

    This is simply a cumulative graph of all requests:

    ![Prometheus Details](/images/prometheus2.png)

When you are finished with Prometheus, terminate the Port Forwarding (Linux: Ctl+C, Mac: Cmd+.)

### Kiali

To start the Port Forwarding issue the following command in a seperate session (and keep it running):

```
$ kubectl port-forward service/kiali 20001:20001 -n istio-system
```

1. In a browser open the Kiali dashboard at http://localhost:20001/
2. Click on the 'Graph' tab, slect the 'default' namespace, and in the 'Display' pulldown, select 'Traffic Distribution'

    ![Kiali UI](/images/kiali1.png)

3. You can now see a graphical representation of your micro services including the distribution of requests amongst your services.
    Watch the distribution of requests amongst the 3 versions the Reviews service: 1/3 = 33.3 % go to each of the versions: equal distribution or "round robin".
    Also note that only v2 and v3 are making requests to the Ratings service.

    ![Kiali Details](/images/kiali2.png)

4. Click on the 'Istio Config' tab:

    ![Kiali Config](/images/kiali3.png)

    Here you can see the Istio specific configuration applied to your microservices. In Exercise 2, step '2 Allow external access to application' you deployed a configuration from file 'samples/bookinfo/networking/bookinfo-gateway.yaml'. This YAML contains the specifications for the Gateway and VirtualService. We will look at hem more closely in the next exercise.


You can keep Kiali and the corresponding Port Forward session open since we will use it frequently.



---

## >> [Continue with Exercise 4](exercise4.md)