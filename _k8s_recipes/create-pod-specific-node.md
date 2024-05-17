---
layout: page
title: Create pod on specific node
permalink: /kubernetes/create-pod-on-specific-node/
parent: ["/kubernetes/", "Kubernetes"]
---

```sh
kubectl run httpd --image httpd --overrides='{"spec":{"nodeName":"k8s-wn2"}}'
```
