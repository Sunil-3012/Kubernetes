# Kubernetes
## Minikube
To create a pod `kubectl run podname`

To get pods `kubectl get pods`

To create a pod via manifest file
```
apiVersion: v1
kind: Pod
metadata:
  name: pod1
spec:
  containers:
    - image: nginx
      name: cont1
```
To list the pods **`kubectl get pods`**

To get inofrmation about pods **`kubectl get pods -o wide`**

To get the detail description about the pod **`kubectl describe pod podname`**

To delete the pod **`kubectl delete pod podname or podID`**

To create a replicaSet
```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: ib-rs
  labels:
    app: bank
spec:
  replicas: 4
  selector:
    matchLabels:
      app: bank
  template:
    metadata:
      labels:
        app: bank
    spec:
      containers:
        - name: cont1
          image: sunil3012/ib-image:latest
```
**`kubectl create -f replicaset.yml`**

To create a deployment 
```

apiVersion: apps/v1
kind: Deployment
metadata:
  name: ib-deployment
  labels:
    app: bank
spec:
  replicas: 3
  selector:
     matchLabels:
       app: bank
  template:
    metadata:
      labels:
        app: bank
    spec:
      containers:
        - name: cont1
          image: sunil3012/ib-image:latest
```
**`kubectl create -f deployment.yml`**

To get the deployments **`kubetctl get deploy`**

To edit the yaml file and update **`kubectl edit deploy deploymentname`**  and save it **`:wq!`**

To scale in and scale out **`kubectl scale deploy deploymentname --replicas=10`**

To describe a pod with a keyword **`kubectl describe pods | grep -i image`**

To delete the deployment **`kubectl delete deployment deploymentname`**



