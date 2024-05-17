---
layout: page
title: Generate kubeadm join command for new worker node
permalink: /kubernetes/generate-kubeadm-join-command/
parent: ["/kubernetes/", "Kubernetes"]
---

```
kubeadm token create --print-join-command
```

then, in the new worker node, run the command.

