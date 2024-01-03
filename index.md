---
layout: default
title: Overview
---

Kubernetes helps managing microservice based applications through self-healing (by minimizing outages and disruptions), intelligent scheduling, horizontal scaling, and load balancing.

[Istio](https://istio.io) is an addition to Kubernetes, also known as a service mesh. It is an open platform to connect, secure, control and observe microservices. With Istio, you can manage network traffic, load balance across microservices, enforce access policies, verify service identity, secure service communication, and observe exactly what is going on with your services.

In this workshop, you will learn how to install and use Istio alongside microservices using the Istio sample application [Bookinfo](https://istio.io/latest/docs/examples/bookinfo/). 

![Bookinfo w/o Istio](images/bookinfo_no_istio.png)

[Minikube](https://minikube.sigs.k8s.io/docs/) will be used to run Kubernetes on your own workstation.

## Objectives 

After you complete this workshop, you’ll be able to:

* Install Istio in your cluster
* Deploy the Bookinfo sample
* Use metrics and tracing to observe services
* Perform simple traffic management, such as A/B tests and canary deployments, and fault injection
* Apply mTLS to your Service Mesh and secure access to services

## Get Started

These are the exercises of this workshop, go through all of them in sequence:

* Exercise 1: [Setting up Minikube and Istio](docs/exercise1)
* Exercise 2: [Installing Bookinfo](docs/exercise2)
* Exercise 3: [Observability](docs/exercise3)
* Exercise 4: [Traffic Management](docs/exercise4)
* Exercise 5: [Traffic Management cont'd](docs/exercise5)

## Software used

This hands-on workshop was created and tested with

* Minikube v1.32.0 (includes Kubernetes v1.28.3) 
* Istio v1.20.1

## Credits

This workshop is based in part on the Istio documentation [Learn Microservices using Kubernetes and Istio](https://istio.io/latest/docs/examples/microservices-istio/) and on my own [Istio Hands-on workshop](https://harald-u.github.io/istio-handson/).
