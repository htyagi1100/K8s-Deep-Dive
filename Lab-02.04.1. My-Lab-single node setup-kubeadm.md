## 4. Kubernetes single node setup using kubeadm tool (CRI_ containerd)

# Step-1 
- Follow this repo
https://github.com/devopsyuva/kubernetes_latest_manifest/tree/main/Kubernetes/01-kubernetes-architecture-Installation

- or 
https://github.com/htyagi1100/K8s-Deep-Dive/blob/main/Lab-02.04-k8s-setup-kubeadm-containerd.md



- Step-1.1 We have to install docker first 
https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository

- Step-1.2 and we can follow this url for kubeadm install
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#installing-kubeadm-kubelet-and-kubectl




- After install packages is reboot files is present on below location then we need to reboot and if this is not present then we no need to reboot 

root@ip-172-31-3-96:~# ls -l /var/run/reboot-required


- Step-1.1 
- if not work then use this
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#installing-kubeadm-kubelet-and-kubectl



- 1. Update the apt package index and install packages needed to use the Kubernetes apt repository:

    sudo apt-get update
    - apt-transport-https may be a dummy package; if so, you can skip that package
    sudo apt-get install -y apt-transport-https ca-certificates curl gpg


- 2. Download the public signing key for the Kubernetes package repositories. The same signing key is used for all repositories so you can disregard the version in the URL:

    curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

- 3. Add the appropriate Kubernetes apt repository.

    echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

- 4. Update the apt package index, install kubelet, kubeadm and kubectl, and pin their version:

    sudo apt-get update
    sudo apt-get install -y kubelet kubeadm kubectl
    sudo apt-mark hold kubelet kubeadm kubectl



# Step-2 

swapoff -a



# Step-3 
```
kubeadm init --apiserver-advertise-address=192.168.1.90 --cri-socket=/run/containerd/containerd.sock --pod-network-cidr=10.244.0.0/16
```



# Step-4 
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```


# Step-5 
- go to this link 
https://docs.tigera.io/calico/latest/getting-started/kubernetes/quickstart

- and under this 
Install Calico
Install the Tigera Calico operator and custom resource definitions.

- There is a yaml file we need to copy and run on kubernetes cluster
```
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.3/manifests/tigera-operator.yaml
```





- Install Calico by creating the necessary custom resource. For more information on configuration options available in this manifest, see the installation reference.

- and second yaml file we don't run direct because we need to changes in this command 
- so we have to download only this link with wget 
```
wget https://raw.githubusercontent.com/projectcalico/calico/v3.26.3/manifests/custom-resources.yaml
```



- now open the file and edit the cidr and give the correct cidr that we have give to our cluster
```
vi custom-resources.yaml

    - blockSize: 26
      cidr: 10.244.0.0/16


- now apply this yaml file 
kubectl apply -f custom-resources.yaml
```



# Step-6 
- now it's not readdy 
```
root@k8s-single:~# kubectl get no
NAME         STATUS     ROLES           AGE   VERSION
k8s-single   NotReady   control-plane   11m   v1.28.3



## Let's verify
root@k8s-single:~# kubectl get po --all-namespaces
NAMESPACE          NAME                                       READY   STATUS    RESTARTS   AGE
calico-apiserver   calico-apiserver-75b48cfb-7nn9j            1/1     Running   0          84s
calico-apiserver   calico-apiserver-75b48cfb-l2nlk            1/1     Running   0          84s
calico-system      calico-kube-controllers-6d6bcfbd86-wqfct   1/1     Running   0          3m51s
calico-system      calico-node-tg748                          1/1     Running   0          3m51s
calico-system      calico-typha-7c95dc96f5-bxmcm              1/1     Running   0          3m52s
calico-system      csi-node-driver-5db4f                      2/2     Running   0          3m51s
kube-system        coredns-5dd5756b68-5f7vr                   1/1     Running   0          15m
kube-system        coredns-5dd5756b68-8x2np                   1/1     Running   0          15m
kube-system        etcd-k8s-single                            1/1     Running   2          15m
kube-system        kube-apiserver-k8s-single                  1/1     Running   2          15m
kube-system        kube-controller-manager-k8s-single         1/1     Running   2          15m
kube-system        kube-proxy-cvmw8                           1/1     Running   0          15m
kube-system        kube-scheduler-k8s-single                  1/1     Running   2          15m
tigera-operator    tigera-operator-597bf4ddf6-np8c4           1/1     Running   0          9m10s






root@k8s-single:~# kubectl get no -o wide
NAME         STATUS   ROLES           AGE   VERSION   INTERNAL-IP       EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
k8s-single   Ready    control-plane   23m   v1.28.3   192.168.192.144   <none>        Ubuntu 22.04.3 LTS   6.2.0-35-generic   containerd://1.6.24




root@k8s-single:~# kubectl get no
NAME         STATUS   ROLES           AGE   VERSION
k8s-single   Ready    control-plane   24m   v1.28.3




# Now try to run a pod, now we got error 
root@k8s-single:~# kubectl run test-pod --image=nginx -t -i
error: timed out waiting for the condition




# Lets verify the issue, describe the pod and check 

root@k8s-single:~# kubectl get po -o wide
NAME       READY   STATUS    RESTARTS   AGE     IP       NODE     NOMINATED NODE   READINESS GATES
test-pod   0/1     Pending   0          2m28s   <none>   <none>   <none>           <none>
root@k8s-single:~# kubectl describe po test-pod
Name:             test-pod
Namespace:        default
Priority:         0
Service Account:  default
Node:             <none>
Labels:           run=test-pod
Annotations:      <none>
Status:           Pending
IP:
IPs:              <none>
Containers:
  test-pod:
    Image:        nginx
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-2b9vd (ro)
Conditions:
  Type           Status
  PodScheduled   False
Volumes:
  kube-api-access-2b9vd:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason            Age    From               Message
  ----     ------            ----   ----               -------
  Warning  FailedScheduling  2m57s  default-scheduler  0/1 nodes are available: 1 node(s) had untolerated taint {node-role.kubernetes.io/control-plane: }. preemption: 0/1 nodes are available: 1 Preemption is not helpful for scheduling..



# now we can see the error, now we have only 1 node that is control plane node so we have to make this compute plan node as well
# Edit the node and remove taints lines 

root@k8s-single:~# kubectl get node
NAME         STATUS   ROLES           AGE   VERSION
k8s-single   Ready    control-plane   44m   v1.28.3

root@k8s-single:~# kubectl edit no k8s-single
node/k8s-single edited

    taints:
  - effect: NoSchedule
    key: node-role.kubernetes.io/control-plane





# Now verify again, pod is running now 

root@k8s-single:~# kubectl get po
NAME       READY   STATUS    RESTARTS   AGE
test-pod   1/1     Running   0          12m


root@k8s-single:~# kubectl get po -o wide
NAME       READY   STATUS    RESTARTS   AGE   IP             NODE         NOMINATED NODE   READINESS GATES
test-pod   1/1     Running   0          12m   10.244.100.7   k8s-single   <none>           <none>


```
