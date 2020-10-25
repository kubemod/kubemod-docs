# Match section

Section `match` is an array of individual criteria items.

When a new object is deployed to Kubernetes, KubeMod intercepts the operation and attempts to match the new object's definition against all ModRules deployed to the namespace where the object is being deployed.

A ModRule is considered to have a match with the Kubernetes object definition when all criteria items in its `match` section yield a positive match.

A criteria item contains a required `select` expression and optional `matchValue`, `matchValues`, `matchRegex` and `negate` properties.

For example, the following `match` section has two criteria items. The `ModRule`  matches all resources whose `kind` is equal to `Deployment` **and** have a container name that's either `container-1` or `container-2` .

```yaml
...
  match:
    - select: '$.kind'
      matchValue: 'Deployment'

    - select: '$.spec.template.spec.containers[*].name'
      matchValues:
        - container-1
        - container-2
```

* `select` - a [JSONPath](https://goessner.net/articles/JsonPath/) expression which, when evaluated against the Kubernetes object definition, yields zero or more values.
* `matchValue` - a string matched against the result of `select`.
* `matchValues` - an array of strings matched against the result of `select`.
* `matchRegex` - a regular expression matched against the result of `select`.

A criteria item is considered positive when its `select` expression yields one or more values and one of the following is true:

* No `matchValue`, `matchValues` or `matchRegex` are specified for the criteria item.
* `matchValue` is specified and one or more of the values resulting from `select` exactly matches that value.
* `matchValues` is specified and one or more of the values resulting from `select` exactly matches one or more of the values in `matchValues`.
* `matchRegex` is specified and one or more of the values resulting from `select` matches that regular expression.

The result of a criteria item can be inverted by setting its `negate` to `true`.

