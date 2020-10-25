# Resource rejection

There are two types of ModRules &mdash; `Patch` and `Reject`.

All of the examples we've seen so far have been of type `Patch`.

`Reject` ModRules are simpler as they only have a `match` section.

If a resource matches the `match` section of a `Reject` ModRule, its creation/update will be rejected.
This enables the development of a system of policy ModRules which enforce certain security restrictions in the namespace they are deployed to.

For example, here's a `Reject` ModRule which rejects the deployment of any `Deployment` or `StatefulSet` resource that does not explicitly require non-root containers:

```yaml
apiVersion: api.kubemod.io/v1beta1
kind: ModRule
metadata:
  name: my-modrule
spec:
  type: Reject

  match:
    # Match (thus reject) Deployments and StatefulSets...
    - select: '$.kind'
      matchValues:
        - 'Deployment'
        - 'StatefulSet'
    # ... that have no explicit runAsNonRoot security context.
    - select: "$.spec.template.spec.securityContext.runAsNonRoot == true"
      negate: true
```
