# Match section

Section `match` of a `ModRule` is an array of individual criteria items.

When a new object is deployed to Kubernetes, KubeMod intercepts the operation and attempts to match the object's definition against all `ModRules` deployed to the namespace where the object is being deployed.

A `ModRule` is considered to have a match with the Kubernetes object definition when all criteria items in its `match` section yield a positive match.

## Criteria item

A criteria item contains a required `select` expression and optional `matchValue`, `matchValues`, `matchRegex` and `negate` fields.

For example, the following `match` section has two criteria items.
The `ModRule`  matches all resources whose `kind` is equal to `Deployment` **and** have a container name that's either `container-1` or `container-2` .

```yaml
...
  match:
    - select: '$.kind'
      matchValue: 'Deployment'

    - select: '$.spec.template.spec.containers[*].name'
      matchValues:
        - 'container-1'
        - 'container-2'
```

A criteria item is considered positive when its `select` expression yields one or more values and one of the following is true:

* No `matchValue`, `matchValues` or `matchRegex` are specified for the criteria item.
* `matchValue` is specified and one or more of the values resulting from `select` exactly matches that value.
* `matchValues` is specified and one or more of the values resulting from `select` exactly matches one or more of the values in `matchValues`.
* `matchRegex` is specified and one or more of the values resulting from `select` matches that regular expression.

The result of a criteria item can be inverted by setting its `negate` to `true`.

A criteria item whose `select` expression yields no results is considered non-matching unless it is `negated`.

### `select` (string : required)

The `select` field of a criteria item is a [JSONPath](https://goessner.net/articles/JsonPath/) expression.

When a `select` expression is evaluated against a Kubernetes object definition, it yields zero or more values.

Let's consider the following JSONPath select expression:

```javascript
$.spec.template.spec.containers[*].name
```

When this expression is evaluated against a `Deployment` resource definition whose specification includes three containers, the result of this `select` expression will be a list of the names of those three containers.

For the purpose of performing matches, KubeMod converts the result of every `select` expression to a list of strings, regardless of what the original type of the target field is.

Here's another example:

```javascript
$.spec.template.spec.containers[*].ports[*].containerPort
```
This expression will yield a list of all `containerPort` values for all ports and all containers. The values in the list will be the string representation of those port numbers.

#### `select` filters

KubeMod `select` expressions includes an extension to JSONPath — filters.

A filter is an expression in the form of `[? <filter expression>]` and can be used in place of JSONPath's `[*]` to filter the elements of an array.

Let's take a look at the following `select` expression:

```javascript
$.spec.template.spec.containers[*].ports[? @.containerPort == 8080]
```

The expression `[? @.containerPort == 8080]` filters the result of the `select` to include only containers that include a port with whose `containerPort` field equals `8080`.

The filter expression could be any JavaScript boolean expression.

The special character `@` represents the current object the filter is iterating over. In the above filter expression, that is the `port` object.

### `matchValue` (string: optional)

When present, the value of field `matchValue` is matched against the results of `select`. If any of the items returned by `select` match `matchValue`, the match criteria is considered a positive match.

The match performed by `matchValue` is case sensitive. If you need case insensitive matches, use `matchRegex`.

### `matchValues` (array of strings: optional)

Field `matchValues` is an array of match values which are tested against the results of `select`. If any of the items returned by `select` match any of the `matchValues` items, the match criteria is considered a positive match.

This match is case sensitive. If you need case insensitive matches, use `matchRegex`.

### `matchRegex` (string: optional)

Field `matchRegex` is a regular expression matched against the results of `select`. If any of the items returned by `select` match `matchRegex`, the match criteria is considered a positive match.

### `negate` (boolean: optional)

Field `negate` can be used to flip the outcome of the criteria item match. Its default value is `false`.
