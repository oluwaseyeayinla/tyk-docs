---
title: "Tyk Helm Chart "
date: 2021-07-01
tags: [""]
description: ""
menu:
  main:
    parent: "Kubernetes "
weight: 1
url: "/tyk-self-managed/tyk-helm-chart"
---

## Introduction

This is the preferred (and easiest) way to install **Tyk Self-Managed** on Kubernetes. 
It will install full Tyk platform with **Tyk Manager**, **Tyk Gateways** and **Tyk Pumps** into your Kubernetes cluster where 
you can add and manage APIs via the **Tyk Operator**, and the **Tyk Manager** (i.e **Tyk Dashboard**).

### Prerequisites

#### 1. Tyk License
If you are evaluating Tyk on Kubernetes, [contact us](https://tyk.io/about/contact/) to obtain a temporary licence.

#### 2. Data stores
The following are required for a Tyk Self-Managed installation:
 - Redis   - Should be installed in the cluster or reachable from inside the cluster (for SaaS option).
             You can find instructions for a simple Redis installation bellow.
 - MongoDB or SQL - Should be installed in the cluster or be reachable by the **Tyk Manager** (for SaaS option).

Installation instructions for Redis and MongoDB/SQL are detailed below.
            
## Installation 

As well as our official Helm repo, you can also find it in [ArtifactHub](https://artifacthub.io/packages/helm/tyk-helm/tyk-pro).
<div class="artifacthub-widget" data-url="https://artifacthub.io/packages/helm/tyk-helm/tyk-pro" data-theme="light" data-header="true" data-responsive="true"><blockquote><p lang="en" dir="ltr"><b>tyk-pro</b>: This chart deploys our full Tyk platform. The Tyk Gateway is a fully open source Enterprise API Gateway, supporting REST, GraphQL, TCP and gRPC protocols. The Tyk Gateway is provided ‘Batteries-included’, with no feature lockout. It enables organisations and businesses around the world to protect, secure, and process APIs and well as review and audit the consumed apis.</p>&mdash; Open in <a href="https://artifacthub.io/packages/helm/tyk-helm/tyk-pro">Artifact Hub</a></blockquote></div><script async src="https://artifacthub.io/artifacthub-widget.js"></script>

If you are interested in contributing to our charts, suggesting changes, creating PRs or any other way, 
please use [GitHub Tyk-helm-chart repo](https://github.com/TykTechnologies/tyk-helm-chart/tree/master/tyk-pro) 
or contact us in [Tyk Community forum](https://community.tyk.io/) or through our sales team.


### Add the Tyk official Helm repo
```bash
helm repo add tyk-helm https://helm.tyk.io/public/helm/charts/
helm repo update
```

### Create namespace for your Tyk deployment
```bash
kubectl create namespace tyk
```

### Getting the values.yaml of the chart
Before we proceed with installation of the chart you need to set some custom values.
To see what options are configurable on a chart and save that options to a custom values.yaml file run:

 ```bash
helm show values tyk-helm/tyk-pro > values.yaml
```

### Installing the data stores
For Redis, MongoDB or SQL you can use these rather excellent charts provided by Bitnami

{{< tabs_start >}}
{{< tab_start "Redis" >}}
<br />
#### Redis
```bash
helm install tyk-redis bitnami/redis -n tyk
```

Follow the notes from the installation output to get connection details and password.

```
  Redis(TM) can be accessed on the following DNS names from within your cluster:

    tyk-redis-master.tyk.svc.cluster.local for read/write operations (port 6379)
    tyk-redis-replicas.tyk.svc.cluster.local for read-only operations (port 6379)

  export REDIS_PASSWORD=$(kubectl get secret --namespace tyk tyk-redis -o jsonpath="{.data.redis-password}" | base64 --decode)
```

The DNS name of your Redis as set by Bitnami is `tyk-redis-master.tyk.svc.cluster.local:6379` (Tyk needs the name including the port)
You can update them in your local `values.yaml` file under `redis.addrs` and `redis.pass`
Alternatively, you can use `--set` flag to set it in Tyk installation. For example  `--set redis.pass=$REDIS_PASSWORD` 
{{< tab_end >}}
{{< tab_start "MongoDB" >}}
<br />
#### MongoDB
```bash
helm install tyk-mongo bitnami/mongodb --set "replicaSet.enabled=true" -n tyk
```

Follow the notes from the installation output to get connection details and password. The DNS name of your MongoDB as set with Bitnami is `tyk-mongo-mongodb.tyk.svc.cluster.local` and you also need to set the `authSource` parameter to `admin`. The full `mongoURL` should be similar to `mongoURL: mongodb://root:pass@tyk-mongo-mongodb.tyk.svc.cluster.local:27017/tyk_analytics?authSource=admin`. You can update them in your local `values.yaml` file under `mongo.mongoURL` Alternatively, you can use `--set` flag to set it in your Tyk installation.

{{< note success >}}
**Important Note regarding MongoDB**

This Helm chart enables the *PodDisruptionBudget* for MongoDB with an arbiter replica-count of 1. If you intend to perform
system maintenance on the node where the MongoDB pod is running and this maintenance requires for the node to be drained,
this action will be prevented due the replica count being 1. Increase the replica count in the helm chart deployment to
a minimum of 2 to remedy this issue.

{{< /note >}}

{{< tab_end >}}
{{< tab_start "SQL" >}}
<br />
**SQL TEXT TO GO HERE**
{{< tab_end >}}
{{< tabs_end >}}

#### Quick Redis and MongoDB PoC installation
{{< warning  success >}}
**Warning**

Another option for Redis and MongoDB, to get started quickly, is to use our **simple-redis** and **simple-mongodb** charts.
Please note that these provided charts must not ever be used in production and for anything
but a quick start evaluation only. Use external redis or Official Redis Helm chart in any other case.
We provide this chart, so you can quickly get up and running, however it is not meant for long term storage of data for example.

```bash
helm install redis tyk-helm/simple-redis -n tyk
helm install mongo tyk-helm/simple-mongodb -n tyk
```
{{< /warning >}}

### License setting

For the **Tyk Self-Managed** chart we need to set the license key in your custom `values.yaml` file under `dash.license` field
or use `--set dash.license={YOUR-LICENSE_KEY}` with the `helm install` command.


Tyk Self-Managed licensing allow for different numbers of Gateway nodes to connect to a single Dashboard instance.
To ensure that your Gateway pods will not scale beyond your license allowance, change the Gateway's resource kind from *DaemonSet* to *Deployment*
and the replica count to your license node limit. For example, use the following options for a single node license:
`--set gateway.kind=Deployment --set gateway.replicaCount=1` in your `values.yaml` file or in the Helm install command.

{{< note success >}}
**Please Note**

There may be intermittent issues on the new pods during the rolling update process, when the total number of online
gateway pods is more than the license limit with lower amounts of Licensed nodes.

{{< /note >}}

### Installing Tyk Self managed
Now we can install the chart using our custom values:

```bash
helm install tyk-pro tyk-helm/tyk-pro -f ./values.yaml -n tyk --wait
```

{{< note success >}}
**Important Note regarding MongoDB**

The `--wait` argument is important to successfully complete the bootstrap of your **Tyk Manager**.

{{< /note >}}

#### Tyk Developer Portal
You can disable the bootstrapping of the Developer Portal by the `portal.bootstrap: false` in your local `values.yaml` file.

#### Using TLS
You can turn on the TLS option under the gateway section in your local `values.yaml` file which will make your Gateway
listen on port 443 and load up a dummy certificate. You can set your own default certificate by replacing the file in the `certs/` folder.

#### Mounting Files
To mount files to any of the Tyk stack components, add the following to the mounts array in the section of that component.
For example:
 ```bash
 - name: aws-mongo-ssl-cert
  filename: rds-combined-ca-bundle.pem
  mountPath: /etc/certs
```

#### Sharding APIs
Sharding is the ability for you to decide which of your APIs are loaded on which of your Tyk Gateways. This option is
turned off by default, however, you can turn it on by updating the `gateway.sharding.enabled` option. Once you do that you
will also need to set the `gateway.sharding.tags` field with the tags that you want that particular Gateway to load. (ex. tags: "external,ingress".)
You can then add those tags to your APIs in the API Designer, under the **Advanced Options** tab, and
the **Segment Tags (Node Segmentation)** section in your Tyk Dashboard.
Check [Tyk Gateway Sharding]({{< ref "/content/advanced-configuration/manage-multiple-environments/manage-multiple-environments.md#api-sharding" >}}) for more details.


## Other Tyk Components

### Installing Tyk Self-managed Control Plane
If you are deploying the **Tyk Control plane**, a.k.a **MDCB**, for a **Tyk Multi Data Centre Bridge** deployment then you set
the `mdcb.enabled: true` option in the local `values.yaml` to add of the **MDCB** component to your installation.
Check [Tyk Control plane](https://tyk.io/docs/tyk-multi-data-centre/) for more configuration details.

This setting enables multi-cluster, multi Data-Centre API management from a single dashboard.


### Tyk Identity Broker (TIB)
The **Tyk Identity Broker** (TIB) is a micro-service portal that provides a bridge between various Identity Management Systems
such as LDAP, OpenID Connect providers and legacy Basic Authentication providers, to your Tyk installation.
See [TIB]({{< ref "/content/tyk-stack/tyk-identity-broker/getting-started.md" >}}) for more details.

For SSO to **Tyk Manager** and **Tyk developer portal** purposes you do not need to install **TIB**, as its functionality is now
part of the **Tyk Manager**. However, if you want to run it separately (as you used to before this merge) or if you need it
 as a broker for the **Tyk Gateway** you can do so.

Once you have installed your **Tyk Gateway** and **Tyk Manager**, you can configure **TIB** by adding its configuration environment variables
under the `tib.extraEnvs` section and updating the `profile.json` in your `configs` folder.
See our [TIB GitHub repo](https://github.com/TykTechnologies/tyk-identity-broker#how-to-configure-tib).
Once you complete your modifications you can run the following command from the root of the repository to update your helm chart.

```bash
helm upgrade tyk-pro values.yaml -n tyk
```

This chart implies there's a **ConfigMap** with a `profiles.json` definition in it. Please use `tib.configMap.profiles` value
to set the name of this **ConfigMap** (`tyk-tib-profiles-conf` by default).

### Tyk as an Ingress using Tyk Operator
To set up an ingress for your Tyk Gateways see our [Tyk Operator GitHub repository](https://github.com/TykTechnologies/tyk-operator).

### Istio Service Mesh with Tyk as an Ingress
To use Tyk's Gateways as the ingress to your Istio Service Mesh simply change `gateway.enableIstioIngress: true` in the
`values.yaml`. Ensure you are using an Istio manifest which disables the default Istio Ingress gateway.


## Next Steps Tutorials
Follow the Tutorials on the **Self-Managed** tabs for the following:

1. [Add an API]({{< ref "/content/getting-started/tutorials/create-api.md" >}})
2. [Create a Security Policy]({{< ref "/content/getting-started/tutorials/create-security-policy.md" >}})
3. [Create an API Key]({{< ref "/content/getting-started/tutorials/create-api-key.md" >}})
