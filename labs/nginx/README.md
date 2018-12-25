Deploying Nginx to our Kubernetes cluster on AWS

(Docker Containers)

# How to proceed deployment on k8s
```
kubectl \
        create deployment my-nginx-deployment \
        --image=nginx
``` 
# How to expose deployment via service

```
kubectl \
        expose deployment my-nginx-deployment \
        --port=80 \
        --type=NodePort \
        --name=my-nginx-service
```
