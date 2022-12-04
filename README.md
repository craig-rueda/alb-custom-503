# Custom 503 Pages With ALB Ingress

A set of example manifests that illustrate a technique for serving up
a custom 503 error page when all backends are unavailable. Possible situations
where this could arise include:

* A misconfiguration causes all pods to become unhealthy
* A planned maintenance of a given app needs to be performed and the replica counts have been temporarily set to `0`

Note that this example is outlined in the linked [blog post](https://medium.com/@crueda/custom-503-page-with-alb-ingress-b2cd652bc018)

## How it works

1. A "maintenance" deployment is created that's sole purpose is to serve up a "site maintenance" page (preferably static  HTML)
   1. The `maintenance` pods ALWAYS report a `HEALTHY` status to kubernetes in order to keep them registered with the ingress/service
   1. The `maintenance` pods ALWAYS report an `UNHEALTHY` status to the associated ALB
1. The `app` pods share the same service with the `maintenance` pods as to include all pods (app + maintenance) in the same target group
1. In the event that all `app` pods become unhealthy, or are scaled down to `0`, [the ALB will forward traffic](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/target-group-health-checks.html) to the `UNHEALTHY` `maintenance` pods

## Prerequisites

* A Kubernetes cluster that you have write access to (running in AWS)
* [ALB Ingress Controller](https://github.com/kubernetes-sigs/aws-load-balancer-controller) installed in your cluster
* **[Optional]** - [external-dns](https://github.com/kubernetes-sigs/external-dns) or some other hostname management service running in your cluster

## Applying

Apply the manifests located in `/manifests`:
```bash
kubectl apply -f ./manifests
```

## Testing

Scale the app pods to `0`
```bash
kubectl scale --replicas=0 deployment/app
# At this point, the maintenance page should be visible

kubectl scale --replicas=2 deployment/app
# At this point, the app should be visible again
```
