---
layout: default
title: 4. Traffic Management 1
---

From the Istio documentation on [Traffic Management](https://istio.io/latest/docs/concepts/traffic-management/):

_Istio’s traffic routing rules let you easily control the flow of traffic and API calls between services. Istio simplifies configuration of service-level properties like **circuit breakers, timeouts, and retries**, and makes it easy to set up important tasks like **A/B testing, canary rollouts, and staged rollouts** with percentage-based traffic splits. It also provides out-of-box **failure recovery features** that help make your application more robust against failures of dependent services or the network._

_Istio’s traffic management model relies on the Envoy proxies that are deployed along with your services. All traffic that your mesh services send and receive (data plane traffic) is proxied through Envoy, making it easy to direct and control traffic around your mesh without making any changes to your services._

## Istio Ingress Gateway 

In Exercise 2, step 2 "Allow external access to application" you have applied an Istio configuration from the file `samples/bookinfo/networking/bookinfo-gateway.yaml`. 

Now we want to look at it. It has 2 parts:

## Part 1: Gateway [&#10162;](https://istio.io/latest/docs/concepts/traffic-management/#gateways) 

The gateway or ingress gateway is a form of load balancer that receives incoming HTTP or TCP connections. It configures exposed ports and protocols and contains the "host" name which is the Internet domain name that the gateway is supposed to accept request for.

Our example:

```
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: bookinfo-gateway
spec:
  selector:
    istio: ingressgateway # use istio default controller
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "*"
```

1. The selector "istio: ingressgateway" maps to the existing istio-ingressgateway deployment in the istio-system namespace. This was created when Istio was installed.
2. The servers section specifies HTTP protocol on port 80 and what is called a "wildcard" host "*". This means that this gateway will direct HTTP requests (unencrypted) on any IP address or domain name. This is actually bad practice but helps keeping this exercise relatively simple.

## Part 2: Virtual Service [&#10162;](https://istio.io/latest/docs/concepts/traffic-management/#virtual-services) 

A Virtual Service is used to configure the Istio traffic routing. Normally, you would use an Istio Virtual Service to add traffic routing rules to a Kubernetes service. Here we use it to configure an Istio Gateway.

Our example:

```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: bookinfo
spec:
  hosts:
  - "*"
  gateways:
  - bookinfo-gateway
  http:
  - match:
    - uri:
        exact: /productpage
    - uri:
        prefix: /static
    - uri:
        exact: /login
    - uri:
        exact: /logout
    - uri:
        prefix: /api/v1/products
    route:
    - destination:
        host: productpage
        port:
          number: 9080
```

The configuration specifies:

* the wildcard ("*") host
* the "bookinfo-gateway" from part 1
* a number of routing rules

To access our Bookinfo sample app in the browser, we use the '/productpage' URI which routes to the productpage deployment on port 9080.

## Destination Rules [&#10162;](https://istio.io/latest/docs/concepts/traffic-management/#destination-rules)

_Along with virtual services, destination rules are a key part of Istio’s traffic routing functionality. You can think of virtual services as how you route your traffic to a given destination, and then you use destination rules to configure what happens to traffic for that destination. Destination rules are applied after virtual service routing rules are evaluated, so they apply to the traffic’s “real” destination._

_In particular, you use destination rules to specify named service subsets, such as grouping all a given service’s instances by version. You can then use these service subsets in the routing rules of virtual services to control the traffic to different instances of your services._

Run the following command to create default destination rules for the Bookinfo services:

```
kubectl apply -f samples/bookinfo/networking/destination-rule-all.yaml
```

Look at the file, specifically at the DestinationRule for 'reviews':

```
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: reviews
spec:
  host: reviews
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
  - name: v3
    labels:
      version: v3
```

Remember that there are 3 versions of the Review service: v1 does not use the Ratings service, v2 and v3 use the Ratings and display stars in black (v2) or red (v3).

Look at the file you used to deploy the Bookinfo application, `samples/bookinfo/platform/kube/bookinfo.yaml`, and there at the section "Reviews service":

There is:
1. a Service definition
2. a ServiceAccount definition
3. three different deployments, labelled "app: reviews" and "version: v1/2/3"

This DestinationRule above defines subsets with names v1, v2, and v3 for the three different deployments (version: v1, v2, or v3).


---

## >> [Continue with Exercise 5](exercise5.md)