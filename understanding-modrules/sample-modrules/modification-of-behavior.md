# Modification of behavior



Here's a typical black-box operator issue which can be fixed with KubeMod: [https://github.com/elastic/cloud-on-k8s/issues/2328](https://github.com/elastic/cloud-on-k8s/issues/2328).

The issue is that when the [Elastic Search operator](https://github.com/elastic/cloud-on-k8s) creates Persistent Volume Claims, it attaches an `ownerReference` to them such that they are garbage-collected after the operator removes the Elastic Search stack of resources.

This makes sense when we plan to dynamically scale Elastic Search up and down, but it doesn't make sense if we don't plan to scale dynamically, but we do want to keep the Elastic Search indexes during Elastic Search reinstallation \(see comments [here](https://github.com/elastic/cloud-on-k8s/issues/2328#issuecomment-583254122) and [here](https://github.com/elastic/cloud-on-k8s/issues/2328#issuecomment-650335893)\).

A solution to this issue would be the following ModRule which simply removes the `ownerReference` from PVCs created by the Elastic Search operator at the time they are deployed, thus excluding those resources from Kubernetes garbage collection:

```text
apiVersion: api.kubemod.io/v1beta1
kind: ModRule
metadata:
  name: my-mod-rule
spec:
  type: Patch

  matches:
    # Match persistent volume claims ...
    - select: '$.kind'
      matchValue: PersistentVolumeClaim
    # ... created by the elasticsearch operator.
    - select: '$.metadata.labels["common.k8s.elastic.co/type"]'
      matchValue: elasticsearch

  patch:
    # Remove the ownerReference if it exists, thus excluding the resource from Kubernetes garbage collection.
    - op: remove
      path: /metadata/ownerReferences/0
```

