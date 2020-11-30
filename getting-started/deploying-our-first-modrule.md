# Deploying our first ModRule

Once KubeMod is installed, you can deploy ModRules to intercept the creation and update of specific resources and perform modifications on them.

For example, here's a ModRule which intercepts the creation of Deployment resources whose `app` labels equal `nginx` and include at least one container of `nginx` version `1.14.*`.

The ModRule patches the matching Deployments on-the-fly to enforce a specific `securityContext` and add annotation `my-annotation`.

Since KubeMod intercepts and patches resources **before** they are deployed to Kubernetes, we are able to patch read-only fields such as `securityContext` without the need to drop and recreate existing resources.

```yaml
apiVersion: api.kubemod.io/v1beta1
kind: ModRule
metadata:
  name: my-modrule
spec:
  type: Patch

  match:
    # Match deployments ...
    - select: '$.kind'
      matchValue: 'Deployment'

    # ... with label app = nginx ...
    - select: '$.metadata.labels.app'
      matchValue: 'nginx'

    # ... and at least one container whose image matches nginx:1.14.* ...
    - select: '$.spec.template.spec.containers[*].image'
      matchRegex: 'nginx:1\.14\..*'

    # ... but has no explicit runAsNonRoot security context.
    # Note: "negate: true" flips the meaning of the match.
    - select: '$.spec.template.spec.securityContext.runAsNonRoot == true'
      negate: true


  patch:
    # Add custom annotation.
    - op: add
      path: /metadata/annotations/my-annotation
      value: whatever

    # Enforce non-root securityContext and make nginx run as user 101.
    - op: add
      path: /spec/template/spec/securityContext
      value: |-
        fsGroup: 101
        runAsGroup: 101
        runAsUser: 101
        runAsNonRoot: true
```

Save the above `ModRule` to file `my-modrule.yaml` and deploy it to the default namespace of your Kubernetes cluster:

```bash
kubectl apply -f my-modrule.yaml
```

After the `ModRule` is created, the creation of any nginx Kubernetes `Deployment` resource in the same namespace will be intercepted by KubeMod, and if the `Deployment` resource matches the ModRule's `match` section, the resource will be patched with the collection of `patch` operations.

## List deployed ModRules

To list all ModRules deployed to a namespace, run the following:

```bash
kubectl get modrules
```

