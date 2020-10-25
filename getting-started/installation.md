---
description: 'Follow these instructions to install, upgrade or uninstall KubeMod'
---

# Installation



## Install

If you are upgrading from a previous version of KubeMod, run the following:

Install KubeMod by running the following command against your Kubernetes cluster:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubemod/kubemod/v0.6.0/bundle.yaml
```

{% hint style="info" %}
The above command will create namespace `kubemod-system` and will deploy KubeMod's resources into it.
{% endhint %}

## Upgrade

If you are upgrading from a previous version of KubeMod, run the following:

```bash
# Delete the kubemod certificate generation job in case kubemod has already been installed.
kubectl delete job -l job-name

# Upgrade kubemod operator.
kubectl apply -f https://raw.githubusercontent.com/kubemod/kubemod/v0.6.0/bundle.yaml
```

## Uninstall

To uninstall KubeMod and all its resources, run:

```text
kubectl delete -f https://raw.githubusercontent.com/kubemod/kubemod/v0.6.0/bundle.yaml
```

{% hint style="warning" %}
Uninstalling KubeMod will also remove all your ModRules deployed to all Kubernetes namespaces.
{% endhint %}

