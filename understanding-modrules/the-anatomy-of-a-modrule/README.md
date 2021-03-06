# The anatomy of a ModRule

A `ModRule` has a `type`, a `match` section, and a `patch` section.

```yaml
apiVersion: api.kubemod.io/v1beta1
kind: ModRule

spec:
  type: ...

  match:
    ...

  patch:
    ...
```

The `type` of a `ModRule` can be one of the following:

* `Patch` — this type of `ModRule` applies patches to objects that match the `match` section of the rule. Section `patch` is required for `Patch` ModRules.
* `Reject` — this type of `ModRule` rejects objects which match the `match` section.

Section [`match`](match-section.md) is an array of individual criteria items used to determine if the `ModRule` applies to a Kubernetes object.

Section [`patch`](patch-section.md) is an array of patch operations.

