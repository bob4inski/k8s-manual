# How to set up  1.28 k8s cluster
[link1](https://computingforgeeks.com/install-kubernetes-cluster-ubuntu-jammy/)

https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/

[link2](https://www.linuxtechi.com/install-kubernetes-on-ubuntu-22-04/)
##  Prepare nodes 

### Remove old docker
```
for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do sudo apt-get remove $pkg; done
```  

### Add docker pkg

```shell
#Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# Add the repository to Apt sources:
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```
### Install Docker
```shell
sudo apt-get install docker-ce docker-ce-cli containerd.io  docker-buildx-plugin docker-compose-plugin
```
containerd очень важно установить

### Disable swap
```shell
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

### 
```shell
sudo tee /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF
sudo modprobe overlay
sudo modprobe br_netfilter
```

###
```shell
sudo tee /etc/sysctl.d/kubernetes.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF 
```

### Configure systemd as cgroup
```shell
containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1
sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml

sudo systemctl restart containerd
sudo systemctl enable containerd
```

##  Kubernetes package repositories
These instructions are for Kubernetes 1.28

```shell
sudo apt-get update
# apt-transport-https may be a dummy package; if so, you can skip that package
sudo apt-get install -y apt-transport-https ca-certificates curl

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

#Add the appropriate Kubernetes apt repository:

# This overwrites any existing configuration in /etc/apt/sources.list.d/kubernetes.list
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
#Update the apt package index, install kubelet, kubeadm and kubectl, and pin their version:

sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```


# Init cluster

Что нужно сделать
1. Проинициализировать кластер 
```shell
sudo kubeadm init --control-plane-endpoint=<ip-address> 
#Заменить ip-address на адрес, по которому будет доступно апи
```
2. join control planes
On 1 master do:
```shell
sudo kubeadm init phase upload-certs --upload-certs
#copy ley
``` 

and 
```shell
kubeadm token create --print-join-command
#copy output commad 
```

then on master-2

```shell
print-join-command --control-plane --certificate-key <key from upload certs>
```


3. 
```shell
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/calico.yaml
```





# links and commands
```shell
sudo rm -rf /var/lib/etcd && sudo rm -rf /var/lib/kubelet/ && rm -rf  $HOME/.kube && sudo rm -rf /etc/cni/ && sudo rm -rf /etc/kubernetes/
#remove all after kubeadm reset
```
- [clean-up](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#tear-down)

- [set-up-kubeadm](https://kubernetes.io/docs/reference/setup-tools/kubeadm/)

- [install-kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)

- [quickstart-calico](https://docs.tigera.io/calico/latest/getting-started/kubernetes/quickstart)

- [Типичные ошибки](https://stackoverflow.com/questions/61305498/kubernetes-couldnt-able-to-join-master-node-error-execution-phase-preflight)

- [ha cluster](https://medium.com/velotio-perspectives/demystifying-high-availability-in-kubernetes-using-kubeadm-3d83ed8c458b#:~:text=High%20Availability%20in%20action,more%20pods%2C%20deployment%20services%20etc.)



# ERRORS
## FileContent--proc-sys-net-bridge-bridge-nf-call-iptables: /proc/sys/net/bridge/bridge-nf-call-iptables does not exist


```bash 
modprobe br_netfilter
sysctl -p /etc/sysctl.conf
```


## [ERROR FileContent--proc-sys-net-ipv4-ip_forward]: /proc/sys/net/ipv4/ip_forward contents are not set to 1
u forgot to do [this](https://kubernetes.io/docs/setup/production-environment/container-runtimes/)

## [ERROR CRI]: container runtime is not running: output: time="2023-09-07T11:16:54+03:00" level=fatal msg="validate service connection: validate CRI v1 runtime API for endpoint \"unix:///var/run/containerd/containerd.sock\": rpc error: code = Unimplemented desc = unknown service runtime.v1.RuntimeService"
```bash
rm /etc/containerd/config.toml
systemctl restart containerd
```









