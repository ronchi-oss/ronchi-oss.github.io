---
layout: page
title: Remove a label from a resource
permalink: /kubernetes/remove-label-from-resource/
parent: ["/kubernetes/", "Kubernetes"]
---

Remove label `foo`:

```sh
kubectl label <resource-type> <resource-name> foo-
```
