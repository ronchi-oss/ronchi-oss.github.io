---
layout: page
title: Describing objects by partial name
permalink: /kubernetes/describe-objects-partial-name/
---
Providing a prefix is enough to match an unambiguous object name. Consider the following output, where pod names    are suffixed with either random strings (regular pods) or the node name (static pods).

```
kubectl get pods -n kube-system | cut -d ' ' -f 1
NAME
calico-kube-controllers-787f445f84-8lhdg
calico-node-54fh5
calico-node-jg9xd
coredns-76f75df574-c2sf6
coredns-76f75df574-lhlwf
etcd-k8s-cp1
kube-apiserver-k8s-cp1
kube-controller-manager-k8s-cp1
kube-proxy-hcxrl
kube-proxy-zbk46
kube-scheduler-k8s-cp1
```

The following commands would work:

```
kubectl describe pod -n kube-system coredns
kubectl describe pod -n kube-system etcd
kubectl describe pod -n kube-system kube-apiserver
```
