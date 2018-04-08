# 1. run this line into your machine console using root permission 

```
curl -sL https://raw.githubusercontent.com/prabhatpankaj/kubernetes-onpremise/master/configure.sh | sh

```
# 2. Create the cluster

At this point we create the cluster by initiating the master with kubeadm. Only do this on the master node.

```
kubeadm init
```
# 3. Take a copy of the Kube config:

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

```

# 4. Make sure you note down the join token command i.e. 

```
kubeadm join 172.31.46.173:6443 --token tw4oxw.wnw97dffmoxkjdg8 --discovery-token-ca-cert-hash sha256:6f0b9f77d03569701f1ce8dfc2ae9753fbc3f31da6f842d---------------

```
# 5. Install networking

```
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"

```
# 6. Join the worker nodes to the cluster
* After finish step 5, you has been completed setup master node of your kubernetes cluster. To setup other machine to join into your cluster
* Prepare your machine as step 1
Run command kubeadm join with params is the secret key of your kubernetes cluser and your master node ip as STEP 4


