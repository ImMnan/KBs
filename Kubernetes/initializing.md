# Starting with Kubernetes

## How to install Kubernetes

1. Kubernetes [documentation](https://kubernetes.io/docs/home/)
2. Installing [Kubernetes](https://kubernetes.io/docs/tasks/tools/)
3. Kubernetes [components](https://kubernetes.io/docs/concepts/overview/components/)

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

Kubelet is a daemon and kub-proxy is a container running on every node in Kubernetes cluster. 
Kube-proxy will configure the network as seen in the following iptables:
```
sudo iptables -S -t nat | grep KUBE
```

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
    sudo kubeadm init | tee kubeadm-init.log
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


## Installing Helm

> For the complete documentation [visit helm](https://helm.sh/docs/intro/install/)

- Helm Package for APT
```
curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
sudo apt-get install apt-transport-https --yes
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm
```

- Helm Package for DNF
```
sudo dnf install helm
```

- Helm Package for brew
```
brew install helm
```

- Helm package for Snap
```
sudo dnf install helm
```

- From Source (Linux, macOS)
```
git clone https://github.com/helm/helm.git
cd helm
make
```

Once you have the Helm Client successfully installed, you can move on to using Helm to manage charts and repo.