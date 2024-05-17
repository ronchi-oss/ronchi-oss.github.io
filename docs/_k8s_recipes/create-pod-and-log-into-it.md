---
layout: page
title: Create pod and log into its container, remove pod on exit
permalink: /kubernetes/create-pod-then-log-into-container/
---

```sh
kubectl run box --image busybox -it --rm
```
