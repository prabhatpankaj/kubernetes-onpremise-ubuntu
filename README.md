# 1. run this line into your machine console 

```
curl -sL https://raw.githubusercontent.com/prabhatpankaj/ubuntustarter/master/initial.sh | sh

curl -sL https://raw.githubusercontent.com/prabhatpankaj/kubernetes-onpremise-ubuntu/master/configure.sh | sh

```
# 2. Running Docker without sudo permits Running Docker with sudo all time is not a great idea. We will fix this in this step 

```
sudo usermod -aG docker ${USER}
newgrp docker
sudo service docker restart

sudo systemctl enable docker.service
```
# 3. Create the cluster

At this point we create the cluster by initiating the master with kubeadm. Only do this on the master node.

```
ifconfig
```
* you should get somthing like this 

```
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 9001
        inet 172.31.63.19  netmask 255.255.240.0  broadcast 172.31.63.255
        inet6 fe80::10f2:f3ff:fe0a:b86e  prefixlen 64  scopeid 0x20<link>
        ether 12:f2:f3:0a:b8:6e  txqueuelen 1000  (Ethernet)
        RX packets 80837  bytes 117959539 (117.9 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 4168  bytes 413343 (413.3 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```
* sipcalc 172.31.63.19/16

```
-[ipv4 : 172.31.63.19/16] - 0

[CIDR]
Host address		- 172.31.63.19
Host address (decimal)	- 2887728915
Host address (hex)	- AC1F3F13
Network address		- 172.31.0.0
Network mask		- 255.255.0.0
Network mask (bits)	- 16
Network mask (hex)	- FFFF0000
Broadcast address	- 172.31.255.255
Cisco wildcard		- 0.0.255.255
Addresses in network	- 65536
Network range		- 172.31.0.0 - 172.31.255.255
Usable range		- 172.31.0.1 - 172.31.255.254

-

```
* We'll now use the internal IP address to broadcast the Kubernetes API - rather than the Internet-facing address.
* You must replace --apiserver-advertise-address with the IP of your host.
```
sudo su

sed -i '9s/^/Environment="KUBELET_EXTRA_ARGS=--fail-swap-on=false"\n/' /etc/systemd/system/kubelet.service.d/10-kubeadm.conf

systemctl daemon-reload

systemctl restart kubelet

kubeadm init --ignore-preflight-errors Swap --pod-network-cidr=172.31.0.0/16 --apiserver-advertise-address=172.31.63.19 --kubernetes-version v1.14.0
```

# 4. Configure an unprivileged user-account and Take a copy of the Kube config:

```
sudo useradd kubeuser -G sudo -m -s /bin/bash
sudo passwd kubeuser
sudo su kubeuser
cd $HOME
sudo cp /etc/kubernetes/admin.conf $HOME/
sudo chown $(id -u):$(id -g) $HOME/admin.conf
export KUBECONFIG=$HOME/admin.conf
echo "export KUBECONFIG=$HOME/admin.conf" | tee -a ~/.bashrc

sudo usermod -aG docker ${USER}
newgrp docker
sudo service docker restart

```

# 5. Make sure you note down the join token command i.e. 

```
kubeadm join 10.0.1.133:6443 --token 0daec3.ql0fin8xr87erlc2 --discovery-token-ca-cert-hash sha256:4a52b12b7953f0713c3a4f4f2084cfad9bc003da12180670a46268589eb1a9d5

```
# 6. Install networking . use Flannel or WeaveWorks
* Flannel provides a software defined network (SDN) using the Linux kernel's overlay and ipvlan modules.

```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/k8s-manifests/kube-flannel-rbac.yml

kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

```
* Another popular SDN offering is Weave Net by WeaveWorks.
```
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"

```

# 7. Join the worker nodes to the cluster
* After finish step 6, you has been completed setup master node of your kubernetes cluster. To setup other machine to join into your cluster
* Prepare your machine as step 1
Run command kubeadm join with params is the secret key of your kubernetes cluser and your master node ip as STEP 4


# 8. Allow a single-host cluster
* Kubernetes is about multi-host clustering - so by default containers cannot run on master nodes in the cluster. Since we only have one node - we'll taint it so that it can run containers for us.
 ```
 kubectl taint nodes --all node-role.kubernetes.io/master-
 ```

# 9. get cluster

```
kubectl get all --namespace=kube-system
```

