# Sidecar injection

With the help of ModRules, one can dynamically inject arbitrary sidecar containers into `Deployment` and `StatefulSet` resources.
The `patch` part of the ModRule is a [Golang template](https://golang.org/pkg/text/template/) which takes the target resource object as an intrinsic context allowing for powerful declarative rules such as the following one which injects a [Jaeger Agent](https://www.jaegertracing.io/docs/1.19/architecture/#agent) sidecar into any Deployment tagged with annotation `my-inject-annotation` set to `"true"`:

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
      matchValue: Deployment
    # ... with annotation  my-inject-annotation = true ...
    - select: '$.metadata.annotations["my-inject-annotation"]'
      matchValue: '"true"'
    # ... but ensure that there isn't already a jaeger-agent container injected in the pod template to avoid adding more containers on UPDATE operations.
    - select: '$.spec.template.spec.containers[*].name'
      matchValue: 'jaeger-agent'
      negate: true

  patch:
    - op: add
      # Use -1 to insert the new container at the end of the containers list.
      path: /spec/template/spec/containers/-1
      value: |-
        name: jaeger-agent
        image: jaegertracing/jaeger-agent:1.18.1
        imagePullPolicy: IfNotPresent
        args:
          - --jaeger.tags=deployment.name={{ .Target.metadata.name }},pod.namespace={{ .Namespace }},pod.id=${POD_ID:},host.ip=${HOST_IP:}
          - --reporter.grpc.host-port=dns:///jaeger-collector-headless.{{ .Namespace }}:14250
          - --reporter.type=grpc
        env:
          - name: POD_ID
            valueFrom:
              fieldRef:
                apiVersion: v1
                fieldPath: metadata.uid
          - name: HOST_IP
            valueFrom:
              fieldRef:
                apiVersion: v1
                fieldPath: status.hostIP
        ports:
        - containerPort: 6832
          name: jg-binary-trft
          protocol: UDP
```

Note the use of ``{{ .Target.metadata.name }}`` in the patch `value` to dynamically access the name of the deployment being patched and pass it to the Jaeger agent as a tracer tag.

When a patch is evaluated, KubeMod executes the patch value as a [Golang template](https://golang.org/pkg/text/template/) and passes the following intrinsic items accessible through the template's context:
* `.Target` - the original resource object being patched with all its properties.
* `.Namespace` - the namespace of the resource object.
