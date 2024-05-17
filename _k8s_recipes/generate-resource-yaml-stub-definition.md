---
layout: page
title: Generate resource YAML stub definition
permalink: /kubernetes/generate-resource-yaml-stub-definition/
parent: ["/kubernetes/", "Kubernetes"]
---

For resources that can be created using `kubectl create`:

```sh
kubectl create <resource> [resource-specific-options] --dry-run=client -o yaml
```

For `Pod` resource:

```sh
kubectl run <pod-name> --image <image> [pod-specific-options] --dry-run=client -o yaml
```

You could run either command from `vim` and have that definition pasted into the file:

```
!!kubectl [...]
```

Since it's not possible to provide any option for overriding further schema properties, we can do that in a         pipeline using `yq`:

```sh
kubectl create service nodeport checkout --tcp 80:80 --dry-run=client -o yaml \
  | yq e '.spec.selector={"run":"nginx"}, .metadata.labels["color"]="red"' - \
  | kubectl apply -f -
```

