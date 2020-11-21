# Debugging ModRules

To list the ModRules deployed to a namespace, run the following command:

```bash
kubectl get modrules
```

When a ModRule does not behave as expected, your best bet is to analyze KubeMod's operator logs.

Follow these steps:

* Deploy your ModRule to the namespace where you will be deploying the Kubernetes resources the ModRule should intercept.
* Find the KubeMod operator pod - run the following command and grab the name of the pod that begins with `kubemod-operator`:

```bash
kubectl get pods -n kubemod-system
```

* Tail the logs of the pod:

```bash
kubectl logs kubemod-operator-xxxxxxxx-xxxx -n kubemod-system -f
```

* In another terminal deploy the Kubernetes object your ModRule is designed to intercept.
* Watch the operator pod logs.

If your ModRule is a Patch rule, KubeMod operator will log the full JSON Patch applied to the target Kubernetes object at the time of interception.

If there are any errors at the time the patch is calculated, you will see them in the logs.

If the operator log is silent at the time you deploy the target object, this means that your ModRule's `match` criteria did not yield a positive match for the target object.

