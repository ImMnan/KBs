# Cluster Administration	

[Kubernetes Cheat sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

- Verify Cluster setup	
- Kubernetes API
- Deep-dive into Master setup
- Registering Working Nodes	
- Deploying the first pod and accessing it
- Working with Kubeadm
- Kubernetes Dashboard
- ETCD - Backup & Restore
- Upgrading Kubernetes Cluster


## Verify Cluster Setup
```
kubectl get nodes 
kubectl config view
```
The config view will show the current-context of the cluster, users etc. example:
```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://172.31.9.25:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
kind: Config
preferences: {}
users:
- name: kubernetes-admin
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED
```

To know more:
```
kubectl config --help
```

To switch the context or the cluster:
```
kubectl config use-context kubernetes-admin@kubernetes
```

## Kubernetes API

Proxy port 8080
```
kubectl proxy --port=8080
```
- Note: Do, CTRL+C to close the proxy. After accessing API Server in the Desktop

Open browser to access the API server http://localhost:8080


## Deep-dive into Master setup

```
kubectl cluster-info
kubectl cluster-info dump > cluster-dump
kubectl get node worker-node-1.example.com
kubectl describe node worker-node1.example.com | less
```
- Look at Status(should be FALSE), Address, Capacity, and Events

```
kubectl get namespaces
kubectl get pods -A
kubectl get pods -n kube-system
```
- Look into /etc/kubernetes/ - Config, manifests & pki/certs

For example to look for manifests file, any changes to manifests will take effect immediately.
```
ls /etc/kubernetes/manifests/
```

- Get a detailed list of the specified resources in a namespace - 
```
kubectl get pods -n kube-system -o wide | grep proxy
service kubelet status
```

## Registering Working Nodes

```
kubectl get nodes
kubectl describe node worker-node1.example.com

kubectl delete node worker-node1.example.com
kubectl get nodes
```

Create a new file with Node info,

```
vi nodereg.json 
```

```json
{
  "kind": "Node",
  "apiVersion": "v1",
  "metadata": {
	"name": "worker-node-1.example.com",
	"labels": {
  	"name": "firstnode"
	}
  }
}

```

Create node:
```
kubectl create -f nodereg.json     
kubectl get nodes
```


## Deploying the first pod and accessing it

```
kubectl run nginxpod --image=nginx --port 80
kubectl get pods
kubectl describe pod nginxpod
kubectl exec -it nginxpod /bin/sh
```


## Working with Kubeadm

Viewing the configuration details
```
kubeadm config print init-defaults
```
Kubernetes certificates
```
sudo kubeadm certs check-expiration
```

## ETCD - Backup

- Step 1: Get URLs and keys
```
kubectl describe pod etcd-master -n kube-system
```
Get client-URL, cert, key, and trusted-ca location

- Step 2: Command
```
sudo snap install etcd
sudo apt install etcd-client
sudo chmod a+rw -R /etc/kubernetes/pki
sudo ETCDCTL_API=3 etcdctl snapshot save etcd_backup.db \
--endpoints https://<cluster-ip>:2379 \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key \
--cacert=/etc/kubernetes/pki/etcd/ca.crt

```

- Step 3: Verify
```
sudo ETCDCTL_API=3 etcdctl --write-out=table snapshot status etcd_backup.db \
--endpoints https://<cluster-ip>:2379 \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key \
--cacert=/etc/kubernetes/pki/etcd/ca.crt
```


## ETCD - Restore

> Works only on multi-master etcd node, Don’t try in single master setup. This command is only for Reference.

```
ETCDCTL_API=3 etcdctl --write-out=table snapshot restore etcd_backup.db \
--endpoints https://<cluster-ip>:2379 \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key \
--cacert=/etc/kubernetes/pki/etcd/ca.crt
```

## Upgrading Kubernetes Cluster

> Verifying the current version of Kubernetes

```
kubeadm version
kubectl get nodes
```

> Upgrading the repositories
```
sudo apt update
sudo apt upgrade
```
If using a RedHat based system, with dnf manager:
```
sudo dnf update
sudo dnf upgrade
```

> Holding the Kubernetes versions
```
sudo apt-mark hold kubeadm
sudo apt-mark hold kubelet kubectl
```

> Upgrading the control plane(Master)
```
sudo apt-get install -y kubeadm=1.23.12-00 --allow-change-held-packages
sudo apt-get install -y kubelet=1.23.12-00 kubectl=1.23.12-00 --allow-change-held-packages
```

> Verifying the updated version of Kubernetes
```
kubeadm version
kubectl get nodes
sudo kubeadm upgrade plan
```