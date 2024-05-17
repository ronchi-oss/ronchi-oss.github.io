---
layout: page
title: Kubernetes development cluster setup
permalink: /kubernetes/kubernetes-development-cluster-setup/
parent: ["/kubernetes/", "Kubernetes"]
---

Using [Canonical Multipass](https://multipass.run) and the [official installation manual](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/) with the [Docker container runtime](https://docs.docker.com/engine/install/ubuntu/).

Create two VMs, one for the control-plane and the other for the worker node. By default Multipass won't assign them the CPU and RAM that kubeadm recommends.

```
multipass launch -n k8s-cp1 -c 2 -m 2G -d 8G
multipass launch -n k8s-cp2 -c 2 -m 2G
multipass launch -n k8s-cp3 -c 2 -m 2G
multipass launch -n k8s-wn1 -c 2 -m 2G -d 8G
multipass launch -n k8s-wn2 -c 2 -m 2G -d 8G
multipass launch -n k8s-lb -c 1 -m 1G
```

Log into the `k8s-cp` box:

```
multipass shell k8s-cp
```

Eventually the control plane VM will run out of disk space. Attempting at resizing it using `multipass set` did not work the last time I needed it (26 Apr 2024). Best is to create the CP VM with 8G of disk.

https://multipass.run/docs/modify-an-instance

Install a container runtime (CRI-O official installation instructions don't work, so we pick Docker):

```
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# check that it works
sudo docker run hello-world
```

Install [cri-dockerd](https://www.mirantis.com/blog/how-to-install-cri-dockerd-and-migrate-nodes-from-dockershim):

```
wget https://github.com/Mirantis/cri-dockerd/releases/download/v0.3.11/cri-dockerd-0.3.11.arm64.tgz
tar xzf cri-dockerd-0.3.11.arm64.tgz
sudo mv cri-dockerd/cri-dockerd /usr/local/bin

# check that it works
cri-dockerd --help

wget https://raw.githubusercontent.com/Mirantis/cri-dockerd/master/packaging/systemd/cri-docker.service
wget https://raw.githubusercontent.com/Mirantis/cri-dockerd/master/packaging/systemd/cri-docker.socket
sudo mv cri-docker.socket cri-docker.service /etc/systemd/system/
sudo sed -i -e 's,/usr/bin/cri-dockerd,/usr/local/bin/cri-dockerd,' /etc/systemd/system/cri-docker.service

sudo systemctl daemon-reload
sudo systemctl enable cri-docker.service
sudo systemctl enable --now cri-docker.socket

# check that it works
sudo systemctl status cri-docker.socket
```

If installing an earlier PATCH version of the latest MINOR version of kubeadm (in order to be able to practice upgrading a cluster), then first it's important to determine what's the earliest PATCH version with:

```
sudo apt list -a kubeadm
```

As of the time of writing (10 Apr 2024) that's 1.29.0-1.1. In that case, the installation command is:

```
KUBE_VERSION='1.29.0-1.1'
sudo apt install -y kubelet="$KUBE_VERSION" kubeadm="$KUBE_VERSION" kubectl="$KUBE_VERSION"
```

Running `apt list` again will show which package has been installed:

```
for package in kubelet kubeadm kubectl; do
    sudo apt list --installed "$package"
done
```

Install kubeadm and run it.

```
sudo apt update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
# if wanting to install latest:
# sudo apt-get install -y kubelet kubeadm kubectl
KUBE_VERSION='1.29.0-1.1'
sudo apt install -y kubelet="$KUBE_VERSION" kubeadm="$KUBE_VERSION" kubectl="$KUBE_VERSION"
sudo apt-mark hold kubelet kubeadm kubectl

sudo systemctl enable --now kubelet
# sudo kubeadm init --pod-network-cidr 192.168.0.0/16 --cri-socket unix:///var/run/cri-dockerd.socket
sudo kubeadm init --pod-network-cidr 192.168.10.0/24 --cri-socket unix:///run/cri-dockerd.sock
```

Optional but recommended if using the `ubuntu` user inside this VM:

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown ubuntu:ubuntu $HOME/.kube/config

kubectl completion bash > kubectl-completion.bash

tee -a $HOME/.bash_profile <<EOF
alias l='ls -lh --color'
export KUBECONFIG=$HOME/.kube/config
source ~/kubectl-completion.bash
EOF
```

For dealing with backup/restore of etcd from the control plane node:

```
sudo apt install -y etcd-client
```

For being able to run kubectl inside the VM as a non-root user:

```
multipass shell k8s-cp # or k8s-wn
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown ubuntu:ubuntu $HOME/.kube/config
```

Install Calico CNI plugin from manifest (not recommeded):

```
curl https://raw.githubusercontent.com/projectcalico/calico/v3.27.3/manifests/calico.yaml -O
kubectl apply -f calico.yaml
```

Log into the worker node VM and run the same commands up until and including the systemctl enable kubelet.

```
multipass shell k8s-wn
# [...]
```

Let the worker node join the cluster by running the `kubeadm join` command provided by the `kubeadm init` command that was executed in the control plane VM. You may need to also provide `--cri-socket`.

Finally, either update your kubectl config file or download it from the control plane node so that you can talk to the cluster from macOS. The following example downloads the entire file to the host:

```
multipass shell k8s-cp
sudo cp /etc/kubernetes/admin.conf .
sudo chown ubuntu:ubuntu admin.conf
exit
scp ubuntu@192.168.64.5:~/admin.conf .
kubectl --kubeconfig ./admin.conf get nodes
```
