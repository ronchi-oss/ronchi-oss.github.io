---
layout: page
title: Share an ephemeral volume between two containers inside same pod
permalink: /kubernetes/share-ephemeral-volume-between-containers-same-pod/
parent: ["/kubernetes/", "Kubernetes"]
---

```yaml
apiVersion: v1
kind: Pod
metadata:
  namespace: default
  name: busybox
spec:
  volumes:
    - name: tmp-vol
      emptyDir: {}
  containers:
  - name: c1
    image: busybox:1.31.1
    command: ["/bin/sh", "-c", "while true; do echo \"$(date '+%Y-%m-%d %H:%M:%S') Hello\" >> /k8s-vol/log.txt; sleep 2; done"]
    volumeMounts:
    - name: tmp-vol
      mountPath: /k8s-vol
  - name: c2
    image: busybox:1.31.1
    command: ["tail", "-f", "/k8s-vol/log.txt"]
    volumeMounts:
    - name: tmp-vol
      mountPath: /k8s-vol
```

