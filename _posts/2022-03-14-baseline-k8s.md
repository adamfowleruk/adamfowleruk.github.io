---
layout: blog
title:  Baseline Kubernetes
date:   2022-03-14 07:00:00 +0000
categories: blog update jekyll level-200
author: adamfowleruk
author_url: https://github.com/adamfowleruk
author_avatar: https://avatars2.githubusercontent.com/u/2700521?s=150&u=7998edeafa7e4a1bf65095b13c8a4fd49c240e84&v=4
---

Every Kubernetes cluster needs a minimum set of services to run. In this post I
talk about installing cert-manager, Istio, and setting up TLS for a basic
workload.

## What we're going to achieve

At the end of this level 200 walkthrough you will be able to:-

- Spin up a new Kuebrnetes cluster in AWS EKS using eksctl
- Add cert-manager using Lets Encrypt's DNS issuing option, with Route 53 for Domain control
- Configure Istio to provide http and https ingress to a sample app
- Enforce mTLS between components within the cluster, and disable raw http (using redirects)
- Install and use Prometheus, Kiali and Grafana to debug Istio configuration problems and visualise traffic in the cluster
- Use basic `istioctl` debug commands

## What you'll need before you proceed

- Access to a Cloud or Private Kubernetes service that is routable from the public internet (for Lets Encrypt validation)
- Access to a domain name (which you will later allocate a subdomain to, pointing at Route 53)

## Creating a cluster

You can use any empty new Kubernetes variant you want. If you don't have one yet,
then follow the below instructions to get one working on Amazon AWS EKS service:-

1. Follow the AWS guide to [install eksctl](https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html)
2. Create a new cluster - I'm using the Ireland region here:-
    ```sh
    eksctl create cluster --region=eu-west-1 blogcluster
    ```
    Now go make yourself a drink as this typically takes around 30 minutes to provision.
3. Verify your connection information:-
    ```sh
    kubectl config get-contexts

    CURRENT   NAME ...
    *         awscli@blogcluster.eu-west-1.eksctl.io ...
    ```

NOTE: If on step 2 above you see an error like `AWS::EC2::EIP/NATIP: CREATE_FAILED – "The maximum number of addresses has been reached...` then you've exceeded the default limit of 5 VPCs per region. Try a different region or ask for a limit increase.

## Add in cert-manager

Cert Manager allows you to plug in various dynamic certificate creation, allocation, and renewal options to your Kubernetes deployment. It's a very commonly used utility throughout Kubernetes.

We're going to instruct Istio to use cert-manager to generate certificates for the subdomain(s) your apps will sit on.

To install cert-manager we're going to use the convenient helm chart:-

```sh
helm repo add jetstack https://charts.jetstack.io
helm repo update
helm install \
  cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.7.1 \
  --set installCRDs=true
```

Note: At the time of writing, v1.7.1 was the latest version.

If this is successful you'll see something like this:-

```text
NOTES:
cert-manager v1.7.1 has been deployed successfully!

In order to begin issuing certificates, you will need to set up a ClusterIssuer
or Issuer resource (for example, by creating a 'letsencrypt-staging' issuer).

More information on the different types of issuers and how to configure them
can be found in our documentation:

https://cert-manager.io/docs/configuration/

For information on how to configure cert-manager to automatically provision
Certificates for Ingress resources, take a look at the `ingress-shim`
documentation:

https://cert-manager.io/docs/usage/ingress/
```

## Install Istio

Istio provides Layer 4 - Layer 7 networking options, and controls Ingress to Kubernetes for HTTP, HTTPS and generic TCP (including TLS) traffic (E.g. for direct connection to RabbitMQ over AMQP or AMQPS).

By default if you wanted to expose an app in Kubernetes you'd set up a load balancer for it. This leads to a proliferation of Load Balancers, and takes time to provision when adding a new app to your cluster. Using Istio instead allows you to create a single load balancer, and serve multiple domains, subdomains, and apps from this one ingress configuration.

I'm going to run you through installing a full Istio set up, using both separate ingress and egress gateways in their own namespaces. This is how you'd do it in production. Many tutorials put everything in the istio-system namespace for convenience so be aware of that in the below instructions compared to other sources.

To install Istio with Prometheus and Kiali to debug it, do the following:-

```sh
helm repo add istio https://istio-release.storage.googleapis.com/charts
helm repo update

kubectl create namespace istio-system

helm install istio-base istio/base -n istio-system

helm install istiod istio/istiod -n istio-system --wait

kubectl create namespace istio-ingress
kubectl label namespace istio-ingress istio-injection=enabled
helm install istio-ingress istio/gateway -n istio-ingress --wait

kubectl create namespace istio-egress
kubectl label namespace istio-egress istio-injection=enabled
helm install istio-egress istio/gateway -n istio-egress --set 'service.type=ClusterIP' --wait

helm status istiod -n istio-system

kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.13/samples/addons/prometheus.yaml

kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.13/samples/addons/kiali.yaml

kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.13/samples/addons/grafana.yaml

```

Note in the above that an egress gateway is just a special type of Istio Gateway that doesn't have an incoming LoadBalancer. It just has an istio proxy (aka Envoy) pod. We'll configure this gateway later to route all egress via it.

## Set up your own domain name using Route53


## Add an app with insecure http ingress

## Visualise traffic routing using Kiali and Grafana

## Configure cert-manager to issue certs using Lets Encrypt

## Add secure https ingress

## Change http to redirect to https

## Force strict mTLS and enable egress filtering for the app namespace


You're done!


## In Summary

You've learnt how to set up a basic Kubernetes service with secure ingress, and how to visualise and debug this using Istio commands and Kiali.

In future parts of the series I'll turn the above into a single helm chart, installable with a single command for convenience!

## Cleaning up

To clear up the entire cluster and remove it, issue the following command:-

```sh
eksctl delete cluster --region=eu-west-1 blogcluster
```

TODO verify the above removes the VPCs too.