# 1. run this line into your machine console 

```
curl -sL https://raw.githubusercontent.com/prabhatpankaj/ubuntustarter/master/initial.sh | sh

curl -sL https://raw.githubusercontent.com/prabhatpankaj/kubernetes-onpremise-ubuntu/master/configure.sh | sh

```
# 1a. Running Docker without sudo permits Running Docker with sudo all time is not a great idea. We will fix this in this step 

```
sudo usermod -aG docker ${USER}
sudo service docker restart
```
# 2. Create the cluster

At this point we create the cluster by initiating the master with kubeadm. Only do this on the master node.

```
ifconfig
```
* you should get somthing like this 

```
eth0      Link encap:Ethernet  HWaddr 02:ac:32:ae:87:20  
          inet addr:10.0.1.133  Bcast:10.0.1.255  Mask:255.255.255.0
          inet6 addr: fe80::ac:32ff:feae:8720/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:9001  Metric:1
          RX packets:18309 errors:0 dropped:0 overruns:0 frame:0
          TX packets:1283 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:26423737 (26.4 MB)  TX bytes:161155 (161.1 KB)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:192 errors:0 dropped:0 overruns:0 frame:0
          TX packets:192 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1 
          RX bytes:14456 (14.4 KB)  TX bytes:14456 (14.4 KB)
```
* sipcalc 10.0.1.133/16

```
-[ipv4 : 10.0.1.133/16] - 0

[CIDR]
Host address		- 10.0.1.133
Host address (decimal)	- 2887715554
Host address (hex)	- AC1F0AE2
Network address		- 10.1.0.0
Network mask		- 255.255.0.0
Network mask (bits)	- 16
Network mask (hex)	- FFFF0000
Broadcast address	- 10.31.255.255
Cisco wildcard		- 0.0.255.255
Addresses in network	- 65536
Network range		- 10.1.0.0 - 10.31.255.255
Usable range		- 10.1.0.1 - 10.31.255.254

-

```
* We'll now use the internal IP address to broadcast the Kubernetes API - rather than the Internet-facing address.
* You must replace --apiserver-advertise-address with the IP of your host.
```
sed -i '9s/^/Environment="KUBELET_EXTRA_ARGS=--fail-swap-on=false"\n/' /etc/systemd/system/kubelet.service.d/10-kubeadm.conf

systemctl daemon-reload

systemctl restart kubelet

kubeadm init --ignore-preflight-errors Swap --pod-network-cidr=10.0.0.0/16 --apiserver-advertise-address=10.0.1.133 --kubernetes-version v1.11.1
```
# 3. add private ip address before joining any slave nodes 
* Adding --node-ip=10.0.1.133 to /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
```
nano /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
```
* restart daemon kubelet as
```
systemctl daemon-reload

systemctl restart kubelet
```
# 4. Configure an unprivileged user-account and Take a copy of the Kube config:

```
sudo useradd kubeuser -G sudo -m -s /bin/bash
sudo passwd kubeuser
sudo su kubeuser
sudo cp /etc/kubernetes/admin.conf $HOME/
sudo chown $(id -u):$(id -g) $HOME/admin.conf
export KUBECONFIG=$HOME/admin.conf
echo "export KUBECONFIG=$HOME/admin.conf" | tee -a ~/.bashrc

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

