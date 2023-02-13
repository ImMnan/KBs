# Cluster Administration	

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
- Look into /etc/kubernetes/ - Config, manifests & pki

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

```
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
--endpoints https://172.31.49.128:2379 \
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