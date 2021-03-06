# Cluster-Config Operator

This operator is able to clone namespaced ConfigMaps and Secrets
to other namespaces. This is a possibility to have cluster-wide
configuration management objects.

## Description

This operator adds two new CustomResourceDefinitions to the Kubernetes API:
* clusterconfigmaps.genesis-mining.com
* clustersecrets.genesis-mining.com

If one of these objects is applied to the cluster, the given ConfigMap
or Secret is applied to all namespaces. If these should not be deployed
to all namespaces, it is possible to include or exclude specific namespaces via names or regex.

Features:
* Cluster-wide deployment of ConfigMaps
* Cluster-wide deployment of Secrets
* Whitelisting of specific Namespaces
* Blacklisting of specific Namespaces

## Installation

### Prerequistes

* Kubernetes 1.16+
* Helm 2.12+

For using the CustomResourceDefinitions, the operator has to be installed first.

### Operator

* Docker Build

```bash
docker build -t yourrepo/clusterconfig .
docker push yourrepo/clusterconfig
```

* Adjust `helm/cluster-config-operator/values.yaml`

* Installing Helm Chart

```bash
cd helm/cluster-config-operator
helm install --name my-release .
```

* Removing the Release

```bash
helm del --purge my-release
```

### ClusterSecrets

To create a ClusterSecret you need to configure a normal secret
in the first step.

To make it cluster-wide available apply a ClusterSecret object:

```yaml
apiVersion: genesis-mining.com/v1beta1
kind: ClusterSecret
metadata:
  name: example-secret
spec:
  name: example-secret
  namespace: default
  excludeNamespaces:
    - ^kube.*
  includeNamespaces:
    - kube-public
```

* Apply the ClusterSecret:

```bash
kubectl apply -f examples/clusterconfigmap.yaml
```

The upstream secret is taken and will be applied to given namespaces.

* Delete the ClusterSecret:
```bash
kubectl delete clustersecret example-secret
```

All applied secrets **except** the upstream secret will be removed.

If a clustersecret is updated (e.g. in includeNamespaces or excludeNamespaces), the changes
will take effect immediatly.

Do not change the settings of the upstream secret.

Path | Type | Explanation
---|---|---
`.spec.name` | String | Name of the cloned secret
`.spec.namespace` | String | Namespace of the cloned secret
`.spec.excludeNamespaces` | Array (String, Regex) | Namespaces which should be blacklisted
`.spec.includeNamespaces` | Array (String, Regex) | Namespaces which should be whitelistet

If includeNamespaces and excludeNamespaces is used at the same time in one object, only includeNamespaces will be deployed.

### ClusterConfigMaps

```yaml
apiVersion: genesis-mining.com/v1beta1
kind: ClusterConfigMap
metadata:
  name: example-configmap
spec:
  name: example-configmap
  namespace: default
  excludeNamespaces:
    - ^kube.*
  includeNamespaces:
    - kube-public
```

* Apply the ClusterConfigMap:

```bash
kubectl apply -f examples/clusterconfigmap.yaml
```

The upstream ConfigMap is taken and will be applied to given namespaces.

* Delete the ClusterConfigMap:
```bash
kubectl delete clusterconfigmap example-configmap
```

All applied ConfigMaps **except** the upstream ConfigMap will be removed.

If a clusterConfigMap is updated (e.g. in includeNamespaces or excludeNamespaces), the changes
will take effect immediatly.

Do not change the settings of the upstream ConfigMap.

If includeNamespaces and excludeNamespaces is used at the same time in one object, only includeNamespaces will be deployed.

## Roadmap

No Roadmap currently

## Contributing

* See [CONTRIBUTING.md](CONTRIBUTING.md)

## Author

* Christopher Becker (@thesolution90)

## Licence

* See [LICENCE](LICENCE)

## Thanks to

* zalando-incubator (https://github.com/zalando-incubator/kopf)
