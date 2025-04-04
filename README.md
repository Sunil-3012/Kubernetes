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
