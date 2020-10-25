# Introduction

## KubeMod

KubeMod is a universal [Kubernetes mutating operator](https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/).

It introduces `ModRule` — a custom Kubernetes resource which allows you to intercept the deployment of any Kubernetes object and apply targeted modifications to it before it is deployed to the cluster.

Use KubeMod to:

* Customize opaque Helm charts and Kubernetes operators
* Build a system of policy rules to reject misbehaving resources
* Develop your own sidecar container injections - no coding required

## Motivation

The creation of KubeMod was motivated by the proliferation of Kubernetes Operators and Helm charts which are sometimes opaque to customizations and lead to runtime issues.

Helm charts and Kubernetes operators greatly simplify the complexity of deploying a ton of primitive resources and reduce it down to a number of configuration values and domain-specific custom resources.

But sometimes this simplicity introduces a challenge — from a user's perspective, Helm charts and Kubernetes operators are black boxes which can only be controlled through the configuration values the chart/operator developer chose to expose.

Ideally we would not need to control anything more than those configuration values, but in reality this opaqueness leads to issues such as these:

* [https://github.com/elastic/cloud-on-k8s/issues/2328](https://github.com/elastic/cloud-on-k8s/issues/2328)
* [https://github.com/jaegertracing/jaeger-operator/issues/1096](https://github.com/jaegertracing/jaeger-operator/issues/1096)

Oftentimes these issues are showstoppers that render the chart/operator impossible to use for certain use cases.

With the help of KubeMod we can make those charts and operators work for us. Just deploy a cleverly developed `ModRule` which targets the problematic primitive resource and patch it on the fly at the time it is created.

