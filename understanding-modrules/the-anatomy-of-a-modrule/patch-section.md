# Patch section

Section `patch` is an array of [RFC6902 JSON Patch](https://tools.ietf.org/html/rfc6902) operations.

The implementation of JSON Patch used in KubeMod includes the following extensions to RFC6902:

* Negative array indices mean starting at the end of the array.
* Operations which attempt to remove a non-existent path in the JSON object are ignored.

In addition, when a patch is evaluated, KubeMod executes the patch `value` as a [Golang template](https://golang.org/pkg/text/template/) and passes the following intrinsic items accessible through the template's context:

* `.Target` - the original resource object being patched with all its properties.
* `.Namespace` - the namespace of the resource object.
