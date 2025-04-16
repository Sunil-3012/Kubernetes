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

## ReplicaSet

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

## Deployment

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

## RollOut commands

To rollout to new image **`kubectl edit deploy/deploymentname`**

To get the history of rollouts **`kubectl rollout history deploy/ib-deployment`**

To rollback to previous version **`kubectl rollout undo deploy/deploymentname`**   


To check if the rollback got executed **`kubectl describe pods | grep -i image`** You will find the previous image

To pause the rollout **`kubectl rollout pause deploy/deploymentname`**  --- it is like lock, cannot undo, cannot rollout to previous

Test it **`kubectl rollout undo deploy/deploymentname`**  -- not possible

To resume it back **`kubectl rollout resume deploy/deploymentname`**

**`kubectl rollout undo deploy/ib-deployment`** -- now possible

To check the status of the rollout **`kubectl rollout status deploy/deploymentname`**

## to create a pod with two containers
```
apiVersion: v1
kind: Pod
metadata:
  name: two-pod-container
  labels:
    app: twopod
spec:
  containers:
    - name: nginxcont
      image: nginx
      ports:
      - containerPort: 95
    - name: busybox-cont
      image: busybox
      command: ["sh","-c", "while true; do echo'hello from busybox'; sleep 10; done"]
```
then **`kubectl apply -f filename`**

To log into a container **`kubectl exec -it podname -c nginxcont -- sh`**  -c indicates container


## Creating a Job

To create a **non-parallel Job**

```
apiVersion: batch/v1
kind: Job
metadata:
  name: non-parallel-job
spec:
  template:
    metadata:
      name: non-parallel-pod
    spec:
      containers:
        - name: non-parallel-container
          image: busybox
          command: ["echo","hello from non parallel world"]
      restartPolicy: Never
  completions: 3
```

To create a **Parallel job**

```
apiVersion: batch/v1
kind: Job
metadata:
  name: parallel-job
spec:
  template:
    metadata:
      name: parallel-pod
    spec:
      containers:
        - name: parallel-cont
          image: busybox
          command: ["echo","hello from parallel world"]
      restartPolicy: Never
  completions: 6
  parallelism: 2
```

**Restart policy can be Never, Always, OnFailure**

**Always**: Always restarts the container if it exits. Use Case: Default for Deployments & ReplicaSets

**OnFailure**: Restarts the container only if it exits with an error (non-zero exit code). Use case: Jobs & CronJobs

**Never**: Never restarts the container, even if it fails. Use Case: Jobs & CronJobs (One-time execution)
