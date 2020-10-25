# Introduction

KubeMod is a universal [Kubernetes mutating operator](https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/).

It introduces `ModRule` — a custom Kubernetes resource which allows you to intercept the deployment of any Kubernetes object and apply targeted modifications to it before it is deployed to the cluster.

Use KubeMod to:

* Customize opaque Helm charts and Kubernetes operators
* Build a system of policy rules to reject misbehaving resources
* Develop your own sidecar container injections - no coding required



