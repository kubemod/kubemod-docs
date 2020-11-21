# Patch section

Section `patch` is an array of [RFC6902 JSON Patch](https://tools.ietf.org/html/rfc6902) operations.

KubeMod's variant of JSON Patch includes the following extensions to RFC6902:

* Negative array indices mean starting at the end of the array.
* Operations which attempt to remove a non-existent path in the JSON object are ignored.

## Patch operation

A patch operation contains fields `op`, `select`, `path` and `value`.

For example, the following `patch` section applies two patch operations executed against every `Deployment` object deployed to the namespace where the `ModRule` resides.

```yaml
...
  match:
    - select: '$.kind'
      matchValue: 'Deployment'

  patch:
    - op: add
      path: /metadata/labels/color
      value: whatever

    # Change all nginx containers' ports from 80 to 8080
    - op: add
      select: $.spec.template.spec.containers[? @.image =~ "nginx" ].ports[? @.containerPort == 80]
      path: /spec/template/spec/containers/#0/ports/#1/containerPort
      value: '8080'
```

The first patch operation adds a new `color` label to the target object.

The second one switches the `containerPort` of all `nginx` containers present in the target `Deployment` object from `80` to `8080`.

Following is a break down of each field of a patch operation.

### `op` \(string: required\)

Field `op` indicates the type of patch operation to be performed against the target object. It can be one of the following:

* `replace` - this type of operation replaces the value of element represented by `path` with the value of field `value`. If `path` points to a non-existent element, the operation fails.
* `add` - this type of operation adds the element represented by `path` with the value of field `value`. If the element already exists, `add` behaves like `replace`.
* `remove` - this type of operation removes the element represented by `path`. If `path` points to a non-existent element, the operation is ignored.

### `select` \(string: optional\) and `path` \(string: required\)

The `select` field of a patch item is a [JSONPath](https://goessner.net/articles/JsonPath/) expression.

When a `select` expression is evaluated against a Kubernetes object definition, it yields zero or more values.

{% hint style="info" %}
For more information about `select` expressions, see [Match item select expressions](match-section.md#select-string-required).
{% endhint %}

When `select` is used in a patch operation, the patch is executed once for each item yielded by `select`.

If the `select` item uses JSONPatch wildcards \(such as `..` or `[*]`\) and/or [filter expressions](match-section.md#select-filtes), KubeMod captures the zero-based index of each wildcard/filter result and makes it available for use in the `path` expression.

Let's consider the following example:

```yaml
select: $.spec.template.spec.containers[*].ports[? @.containerPort == 80]
path: /spec/template/spec/containers/#0/ports/#1/containerPort
value: '8080'
```

The `select` expression includes a wildcard to loop over all containers \(`containers[*]`\), and then a filter \(`ports[? @.containerPort == 80]`\) to select only the ports whose `containerPort` is equal to `80`.

If we evaluate this against a `Deployment` with the following four containers and ports...

```yaml
...
  containers:
    - name: c1
      ports:
      - containerPort: 100
      - containerPort: 200

    - name: c2
      ports:
      - containerPort: 100
      - containerPort: 80

    - name: c3
      ports:
      - containerPort: 100
      - containerPort: 200

    - name: c4
      ports:
      - containerPort: 80
      - containerPort: 200
      - containerPort: 300
...
```

... the `select` expression will yield the following two items:

* Item 1: The second port of the second container
* Item 2: The first port of the fourth container

The zero-based wildcard/filter indexes captured by KubeMod for this `select` will be as follows:

* Item 1:
  * container index: 1
  * port index: 1
* Item 2:
  * container index: 3
  * port index: 0

The indexes above can be used when constructing the `path` of the patch operation.

For example:

```yaml
path: /spec/template/spec/containers/#0/ports/#1/containerPort
```

Note the index placeholders `#0` and `#1`. Each placeholder refers to an index captured by KubeMod when evaluating the `select` expression.

In our previous `select` example, `#0` would be the placeholder for the container index, and `#1` would be the placeholder for the port index.

When KubeMod performs patch operations, it constructs the `path` by replacing the index placeholders with the value of the corresponding index.

For the above `select` and `path` examples executed against our sample `Deployment`, KubeMod will generate two patch operations which will target the following paths:

* `/spec/template/spec/containers/1/ports/1/containerPort`
* `/spec/template/spec/containers/3/ports/0/containerPort`

Combining `path` expressions with index placeholders gives us the ability to perform sophisticated targeted resource modifications.

If `select` is not specified, `path` is rendered as-is and is not subject to index placeholder interpolation.

### value \(string\)

`value` is required for `add` and `replace` operations.

`value` is the **string representation** of a `YAML` value. It can be a primitive value or a complex `YAML` object or array.

Here are a few examples:

Number \(note the quotes - `value` itself is a string, but its "value" evaluates to a `YAML` number\):

```yaml
value: '8080'
```

String:

```yaml
value: hello
```

or

```yaml
value: 'hello'
```

String representation of a number \(note the double-quotes\):

```yaml
value: '"8080"'
```

Boolean:

```yaml
value: 'false'
```

String representation of a boolean:

```yaml
value: '"false"'
```

A complex `YAML` object:

```yaml
value: |-
  name: jaeger-agent
  image: jaegertracing/jaeger-agent:1.18.1
  imagePullPolicy: IfNotPresent
  ports:
  - containerPort: 6832
    name: jg-binary-trft
    protocol: UDP
```

#### Golang Template

When `value` contains `{{ ... }}`, it is evaluated as a [Golang template](https://golang.org/pkg/text/template/).

The following intrinsic items accessible through the template's context:

* `.Target` — the original resource object being patched.
* `.Namespace` — the namespace of the resource object being patched.

For example, the following excerpt of a Jaeger side-car injection `ModRule` includes a `value` which uses `{{ .Target.metadata.name }}` to access the name of the `Deployment` being patched:

```yaml
...
value: |-
  name: jaeger-agent
  image: jaegertracing/jaeger-agent:1.18.1
  imagePullPolicy: IfNotPresent
  args:
    - --jaeger.tags=deployment.name={{ .Target.metadata.name }}
  ports:
  - containerPort: 6832
    name: jg-binary-trft
    protocol: UDP
...
```

