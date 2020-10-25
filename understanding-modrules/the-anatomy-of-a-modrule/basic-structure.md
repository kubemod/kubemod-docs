# Basic structure

A `ModRule` has a `type`, a required `match` section and an optional `patch` section.

```yaml
apiVersion: api.kubemod.io/v1beta1
kind: ModRule

spec:
  type:

  match:

  patch:

```

The `type` of a `ModRule` can be one of the following:
- `Patch` - this type of `ModRule` applies patches to objects that match the `match` section of the rule. Section `patch` is required for `Patch` ModRules.
- `Reject` - this type of `ModRule` rejects objects which match the `match` section.

Section `match` is an array of individual criteria items.

Section `patch` is an array of patch operations.
