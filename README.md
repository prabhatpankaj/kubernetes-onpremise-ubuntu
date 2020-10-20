# 1. run this line into your machine console 

```
curl -sL https://raw.githubusercontent.com/prabhatpankaj/ubuntustarter/master/initial.sh | sh

curl -sL https://raw.githubusercontent.com/prabhatpankaj/kubernetes-onpremise-ubuntu/master/configure.sh | sh

```
# 2. Running Docker without sudo permits Running Docker with sudo all time is not a great idea. We will fix this in this step 

```
sudo usermod -aG docker ${USER}
newgrp docker

```
# 3. At this point we create the cluster by initiating the master with kubeadm. Only do this on the master node.
Initialize the cluster (Execute the following command only on the Master node)
Note: The parameter pod-network-cidr changes as per the network option.
Example: The suggested CIDR for flannel and canal networks is 10.244.0.0/16 and for calico network it could be 192.168.0.0/16.

```
sudo kubeadm init --pod-network-cidr=192.168.0.0/16

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
source ~/.bashrc

sudo usermod -aG docker ${USER}
newgrp docker
sudo service docker restart

```

# 5. Make sure you note down the join token command i.e. 

```
kubeadm join 10.0.1.133:6443 --token 0daec3.ql0fin8xr87erlc2 --discovery-token-ca-cert-hash sha256:4a52b12b7953f0713c3a4f4f2084cfad9bc003da12180670a46268589eb1a9d5

```
# 6. Install Calico
```
kubectl create -f https://docs.projectcalico.org/manifests/tigera-operator.yaml
kubectl create -f https://docs.projectcalico.org/manifests/custom-resources.yaml


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

