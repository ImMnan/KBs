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

