---
layout: page
title: Drain a node
permalink: /kubernetes/drain-node/
---

Watch it on a terminal:

```
kubectl get pods -A --field-selector spec.nodeName=<nodeName> -w
```

Drain it:

```
kubectl drain <nodeName> --ignore-daemonsets --delete-emptydir-data --disable-eviction
```

