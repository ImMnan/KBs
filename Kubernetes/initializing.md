# Starting with Kubernetes

## How to install Kubernetes

1. https://kubernetes.io/docs/setup/
2. https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/
3. https://kubernetes.io/docs/concepts/overview/components/

The pod network will have the following address:
```
192.168.0.0/16
```
That means a total number of 2^16 pods per cluster:
```
2^16 = 64k pods
```
The actual maximum number of pods per worker node is approximately 100 pods.
If you have 300 worker nodes that will do:
```
300 worker x 100 pod/worker = 30k pods
```
The host prefix:
* The subnet prefix length to assign to each individual node. For example, if hostPrefix is set to 24 then each node is assigned a /24 subnet out of the given cidr. A hostPrefix default value of 24 provides 2^(32 - hostPrefix) = 2^(32-24) = 256 pod IP addresses.

For example:
* Worker node1: 192.168.1.0/24
* Worker node2: 192.168.2.0/24
* Worker node3: 192.168.3.0/24

If the default number of IP addresses per worker node is 256, then where does the recommended number of 100 pods per worker come from?
* Because Kubernetes (by default) will never delete an old replica pod until the new replica pod is ready. In the meantime there will always be two replica pods running at the same time (until the new one is ready and the old one is deleted).


## Kubernetes Architecture

> Master Node

- API Server
- Scheduler
- Controller Manager
- Cloud Controller Manager(CCM)
- Etcd

> Worker Nodes
- Kubelet <-> Dockerd
- Kube-proxy
- Pods - Containers - App running
- Operator - Access API-Server for k8s calls - Create, Update, Read, Delete resources
- K8s client - UI, API, CLI - kubectl
- User - App users - Access App deployed in k8s



## Docker Configuration - Master & worker nodes

Process the following steps in Master, Worker1 and Worker2 nodes

```
sudo mkdir /etc/docker
```

```
cat <<EOF | sudo tee /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
	"max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF
```

```
sudo systemctl enable docker
sudo systemctl daemon-reload
sudo systemctl restart docker
sudo swapoff -a
```


## Kubernetes installation procedure
1. Check that Kubernetes components are already installed:
    ```
    service kubelet status
    which kubeadm
    which kubectl
    ls /etc/kubernetes/manifests/
    ```

2. Remove any previous Kubernetes initialization:    
    ```
    sudo kubeadm reset --force
    ```

3. Initialize Kubernetes:

    ```   
    sudo kubeadm init --pod-network-cidr 192.168.0.0/16 --ignore-preflight-errors all 2>& 1 | tee kubeadm-init.log
    ```

4. Check the static pods that have been created for the Control Plane:
    ```    
    ls /etc/kubernetes/manifests/
    ls -l /etc/kubernetes/admin.conf
    ```

5. Execute the following commands to avoid the need of running Kubernetes commands as root:
    ```
    mkdir -p ${HOME}/.kube
    sudo cp /etc/kubernetes/admin.conf ${HOME}/.kube/config
    sudo chown -R $( id -u ):$( id -g ) ${HOME}/.kube/
    echo 'source <(kubectl completion bash)' | tee --append ${HOME}/.bashrc
    source ${HOME}/.bashrc
    ```

6. Deploy a network:    
    ```
    kubectl apply --filename https://docs.projectcalico.org/v3.21/manifests/calico.yaml
    kubectl get no -o wide w 
    ```

7. Create a token to add more workers to the cluster:    
    ```
    sudo kubeadm token create --print-join-command
    ```
