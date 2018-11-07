### minikube deploy

Create a deployment
```bash
kubectl run hello-node --image=hello-node:v1 --port=8080
```

Expose as a service


Get nodeport from minikube
```bash
minikube service hello-node --url
``` 

