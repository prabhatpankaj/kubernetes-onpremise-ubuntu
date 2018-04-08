# run this line into your machine console using root permission 

```
curl -sL https://gist.githubusercontent.com/prabhatpankaj/5b3347913946a2cea04c644bec941a4a/raw/8e692f05e2f3bcc1a2cc33df562a55aa332a07b2/configure.sh | sh

```
# Create the cluster

At this point we create the cluster by initiating the master with kubeadm. Only do this on the master node.

```
kubeadm init
```
# Take a copy of the Kube config:

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

```

# Make sure you note down the join token command i.e. 

```
kubeadm join 172.31.46.173:6443 --token tw4oxw.wnw97dffmoxkjdg8 --discovery-token-ca-cert-hash sha256:6f0b9f77d03569701f1ce8dfc2ae9753fbc3f31da6f842d---------------

```
# Install networking

```
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"

```
# Join the worker nodes to the cluster
