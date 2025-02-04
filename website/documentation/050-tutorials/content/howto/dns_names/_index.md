---
title: Request DNS Names
description: "Requesting DNS Names for Ingresses and Services in Shoot Clusters"
type: tutorial-page
level: Advanced
index: 10
category: DNS
scope: developer
---

# Request DNS Names in Shoot Clusters

## Introduction
Gardener allows Shoot clusters to request DNS names for Ingresses and Services out of the box. 
Therefore the gardener must be installed with the `shoot-dns-service` extension.
This extension uses the seed's dns management infrastructure to maintain DNS
names for shoot clusters. So, far only the external DNS domain of a shoot
(already used for the kubernetes api server and ingress DNS names) can be used
for managed DNS names.

<style>
#body-inner blockquote {
    border: 0;
    padding: 10px;
    margin-top: 40px;
    margin-bottom: 40px;
    border-radius: 4px;
    background-color: rgba(0,0,0,0.05);
    box-shadow: 0 3px 6px rgba(0,0,0,0.16), 0 3px 6px rgba(0,0,0,0.23);
    position:relative;
    padding-left:60px;
}
#body-inner blockquote:before {
    content: "!";
    font-weight: bold;
    position: absolute;
    top: 0;
    bottom: 0;
    left: 0;
    background-color: #00a273;
    color: white;
    vertical-align: middle;
    margin: auto;
    width: 36px;
    font-size: 30px;
    text-align: center;
}
</style>

## Shoot Feature Gate

The shoot DNS feature is not globally enabled by default (depends on the 
extension registration on the garden cluster). Therefore it must be
enabled per shoot.

To enable the feature for a shoot, the shoot manifest must add the `shoot-dns-service` extension.

```yaml
...
spec:
  extensions:
    - type: shoot-dns-service
...
```

## Configuration In Shoot Cluster

To request a DNS name for an Ingress or Service object in the shoot cluster
it must be annotated with the DNS class `garden` and an annotation denoting
the desired DNS names.

For a Service (it must have the type `LoadBalancer`) this looks like this:

```yaml
apiVersion: v1
kind: Service
metadata:
  annotations:
    dns.gardener.cloud/class: garden
    dns.gardener.cloud/dnsnames: my.subdomain.for.shootsomain.cloud
  name: my-service
  namespace: default
spec:
  ports:
    - port: 80
      protocol: TCP
      targetPort: 80
  type: LoadBalancer
```

The *dnsnames* annotation accepts a comma-separated list of DNS names, if
multiple names are required.

For an Ingress, the dns names are already declared in the specification.
Nevertheless the *dnsnames* annotation must be present. Here a subset of the 
dns names of the ingress can be specified. If DNS names for all names are
desired, the value `all` can be used.

If one of the accepted dns names is a direct subname of the shoot's ingress
domain, this is already handled by the standard wildcard entry for the ingress
domain. Therefore this name should be excluded from the *dnsnames* list in the
annotation. If only this dns name is configured in the ingress, no explicit 
dns entry is required, and the dns annotations should be omitted at all.
