---
description: 'Follow these instructions to install, upgrade or uninstall KubeMod'
---

# Installation

## Install

As a Kubernetes operator, KubeMod is deployed into its own namespace — `kubemod-system`.  
The following command will create namespace `kubemod-system` and will deploy KubeMod into it.

```bash
kubectl apply -f https://raw.githubusercontent.com/kubemod/kubemod/v0.7.1/bundle.yaml
```

## Upgrade

If you are upgrading from a previous version of KubeMod, run the following:

```bash
# Delete the kubemod certificate generation job in case KubeMod has already been installed.
kubectl delete job -l job-name -n kubemod-system

# Upgrade KubeMod operator.
kubectl apply -f https://raw.githubusercontent.com/kubemod/kubemod/v0.7.1/bundle.yaml
```

## Uninstall

To uninstall KubeMod and all its resources, run:

```text
kubectl delete -f https://raw.githubusercontent.com/kubemod/kubemod/v0.7.1/bundle.yaml
```

{% hint style="warning" %}
Uninstalling KubeMod will also remove all your ModRules deployed to all Kubernetes namespaces.
{% endhint %}

