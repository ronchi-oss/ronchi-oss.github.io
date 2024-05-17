---
layout: page
title: Create pod and override its command
permalink: /kubernete/create-pod-override-command/
parent: ["/kubernetes/", "Kubernetes"]
---

```sh
kubectl run box --image busybox --command -- /bin/sh -c 'while true; do echo Hello; sleep 1; done'
```

