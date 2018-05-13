# 1. run this line into your machine console using root permission 

```
curl -sL https://raw.githubusercontent.com/prabhatpankaj/ubuntustarter/master/initial.sh | sh

curl -sL https://raw.githubusercontent.com/prabhatpankaj/kubernetes-onpremise/master/configure.sh | sh

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
* We'll now use the internal IP address to broadcast the Kubernetes API - rather than the Internet-facing address.
* You must replace --apiserver-advertise-address with the IP of your host.
```
kubeadm init --pod-network-cidr=10.0.0.0/16 --apiserver-advertise-address=10.0.1.133 --kubernetes-version stable-1.10
```
# 3. Configure an unprivileged user-account and Take a copy of the Kube config:

```
sudo useradd kubeuser -G sudo -m -s /bin/bash
sudo passwd kubeuser
sudo su kubeuser
sudo cp /etc/kubernetes/admin.conf $HOME/
sudo chown $(id -u):$(id -g) $HOME/admin.conf
export KUBECONFIG=$HOME/admin.conf
echo "export KUBECONFIG=$HOME/admin.conf" | tee -a ~/.bashrc

```

# 4. Make sure you note down the join token command i.e. 

```
kubeadm join 10.0.1.133:6443 --token 0daec3.ql0fin8xr87erlc2 --discovery-token-ca-cert-hash sha256:4a52b12b7953f0713c3a4f4f2084cfad9bc003da12180670a46268589eb1a9d5

```
# 5. Install networking . use Flannel or WeaveWorks
* Flannel provides a software defined network (SDN) using the Linux kernel's overlay and ipvlan modules.

```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/k8s-manifests/kube-flannel-rbac.yml
```
* Another popular SDN offering is Weave Net by WeaveWorks.
```
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"

```

# 6. Join the worker nodes to the cluster
* After finish step 5, you has been completed setup master node of your kubernetes cluster. To setup other machine to join into your cluster
* Prepare your machine as step 1
Run command kubeadm join with params is the secret key of your kubernetes cluser and your master node ip as STEP 4


