# Common use cases

## Motivation

The development of KubeMod was motivated by the proliferation of Kubernetes Operators and Helm charts which are sometimes opaque to customizations and lead to runtime issues.

Helm charts and Kubernetes operators greatly simplify the complexity of deploying a ton of primitive resources and reduce it down to a number of configuration values and domain-specific custom resources.

But sometimes this simplicity introduces a challenge — from a user's perspective, Helm charts and Kubernetes operators are black boxes which can only be controlled through the configuration values the chart/operator developer chose to expose.

Ideally we would not need to control anything more than those configuration values, but in reality this opaqueness leads to issues such as these:

* [https://github.com/elastic/cloud-on-k8s/issues/2328](https://github.com/elastic/cloud-on-k8s/issues/2328)
* [https://github.com/jaegertracing/jaeger-operator/issues/1096](https://github.com/jaegertracing/jaeger-operator/issues/1096)

Oftentimes these issues are showstoppers that render the chart/operator impossible to use for certain use cases.

With the help of KubeMod we can make those charts and operators work for us. Just deploy a cleverly developed `ModRule` which targets the problematic primitive resource and patch it on the fly at the time it is created.

See the next sections for a number of typical use cases for KubeMod.

