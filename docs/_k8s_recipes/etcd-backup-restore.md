---
layout: page
title: Back up and restore etcd database
permalink: /kubernetes/backup-restore-etcd/
---

etcd is meant to be accessed through an API, likely following the Kubernetes API server model. Instead of sending   `curl` requests to a URL though, an official client, `etcdctl` should be installed in the control plane node. Each  call to the etcd API through `etcdctl` needs to (1) refer to version 3 of the API, which causes a different         argument list to be required and (2) include the three keys in order to authenticate properly:

```
sudo ETCDCTL_API=3 etcdctl \
    --cacert /etc/kubernetes/pki/etcd/ca.crt \
    --cert /etc/kubernetes/pki/etcd/server.crt \
    --key /etc/kubernetes/pki/etcd/server.key \
    snapshot save etcd-snapshot.db
```

In order to know which auth parameters to use:

```
sudo ETCDCTL_API=3 etcdctl help
```

The path to these files is part of the command-line arguments to the etcd process started by the etcd pod:

```
kubectl describe pod etcd -n kube-system
```

The backup and restore process consists of (from the control plane node):

* Create a DB snapshot file
* Restore it in a new directory in order to create the `member` file
* Stop the kubelet and docker services
* If restoring the database from a different path, edit the etcd static pod manifest under `/etc/kubernetes/        manifest/etcd.yaml` and replace all occurrences of `/var/lib/etcd` for the new directory. Else, overwrite `/var/lib/etcd/member` with the snapshot you want to restore from.
* Start the kubelet and docker services

```
sudo ETCDCTL_API=3 etcdctl \
    --cacert /etc/kubernetes/pki/etcd/ca.crt \
    --cert /etc/kubernetes/pki/etcd/server.crt \
    --key /etc/kubernetes/pki/etcd/server.key \
    snapshot save etcd-snapshot.db
sudo ETCDCTL_API=3 etcdctl --data-dir <path> snapshot restore etcd-snapshot.db
sudo systemctl stop kubelet docker
# either edit /etc/kubernetes/manifest/etcd.yaml or replace /var/lib/etcd/member with desired snapshot
sudo systemctl start kubelet docker
kubectl get pods -A -n kube-system | grep etcd
# it should show status 1/1 Running
```

