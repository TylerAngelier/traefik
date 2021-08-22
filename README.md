# Traefik

As with the [Philosophy of the Official Traefik Helm Chart](https://github.com/traefik/traefik-helm-chart/tree/master/traefik#philosophy) this repository uses the first option, overriding default configuration via a yaml file.

This [Traefik](https://doc.traefik.io/traefik/) install is exposed via a NodePort and therefore requires some type of LoadBalancer in front of it. I am using [OCI](https://www.oracle.com/cloud/)'s [OKE](https://www.oracle.com/cloud-native/container-engine-kubernetes/) with an [OCI Load Balancer](https://www.oracle.com/cloud/networking/load-balancing/). I have a backend set for the HTTP nodePort and can configure listeners as needed. I also have configured my DNS with [Cloudflare](https://www.cloudflare.com) so I am using a wildcard [Origin Certificate](https://www.cloudflare.com/ssl/) from Cloudflare (doesn't expire for 15 years!!) as the SSL cert for the OCI Load Balancer. This means I can configure any domains in Cloudflare and in the OCI Load Balancer and SSL just works! Combine this with Traefik and I can easily deploy services and reach them with SSL.

# Install & Deployment

## Prerequisites

Follow all of the [Traefik Prerequisites](https://github.com/traefik/traefik-helm-chart/tree/master/traefik#prerequisites).


Create Namespace for Traefik.
```
helm create ns traefik
```

## Environment

Traefik Pilot is a neat free resource to use to get Alert, Metrics and Plugins to your Traefik instance. This does require a token for authenticating your Traefik service. See the [Traefik Pilot](#-Traefik-Pilot) section for more information.

If this is your first setup, copy the example `.env` file.
```
cp .env.example .env
```
Add your token to the `.env`. file. Source the file to set your environment variables.
```
source .env
```

## Install

Use the default helm chart that Traefik provides with some overridden values.
```
helm install --namespace=traefik traefik traefik/traefik --values=values.yaml
```

To port forward the Traefik Dashboard grab a pod:
```
kubectl port-forward <pod-name> 9000:9000
```

## Upgrade

After changes, if you need to upgrade the helm chart deployment:
```
helm upgrade -f values.yaml traefik traefik/traefik
```


## OCI

Create an OKE Stack.

Create an OCI Load Balancer.

Configure backend set to the NodePort (in values.yaml).

Create and add Cloudflare Origin Certificate to OCI certificates.

Create desired hostname.

Configure listener to backend set created above.

## Authentication & Authorization

TODO

## Example Service

There is an example `whoami` Service example in the `/examples` directory. The first file is the Kubernetes Deployment and Service - nothing special here. The second file is the interesting tidbit - by simply declaring an `IngressRoute` with a `kind: Rule` we can tell Traefik how to route traffic to our service. This example will route any traffic with the path `/whoami` to our service!

```
kubectl apply -f examples/whoami-example-deployment.yaml
kubectl apply -f examples/whoami-example-ingressroute.yaml
```

# Traefik Pilot

Traefik Pilot is a software-as-a-service (SaaS) platform that connects to Traefik to extend its capabilities. It offers a number of features to enhance observability and control of Traefik through a global control plane and dashboard, including:

* Metrics for network activity of Traefik proxies and groups of proxies
* Alerts for service health issues and security vulnerabilities
* Plugins that extend the functionality of Traefik

Getting this working is very easy. Just follow the [Traefik Pilot Quick Start Guide](https://doc.traefik.io/traefik-pilot/connecting/).

TLDR: Register for service, Register New Traefik Instance, grab token.

# Resources

* [Traefik Helm Chart](https://github.com/traefik/traefik-helm-chart/tree/master/traefik)
* [Official Helm Chart Values](https://github.com/traefik/traefik-helm-chart/blob/master/traefik/values.yaml)
* [Traefik Pilot](https://pilot.traefik.io)
* [Traefik Pilot Docs](https://doc.traefik.io/traefik/plugins/)