# Kubernetes

Topics:

1)Minikube

2)replicaSet

3)Deployment

4)Creating a Job

5)Creating a cluster

6)Creating Namspaces

7)Creating users,roles and permissions


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

## Creating a cluster (1 master node and 2 worker nodes)

Fist configure AWS via CLI

First create a s3 bucket where all the data about the cluster can be stored

```
aws s3api create-bucket --bucket sunil-kops-testbkt.k8s.local --region us-east-1

```

To craete a bucket in another regrion other than us-east-1 you need to use

```
aws s3api create-bucket --bucket sunil-kops-testbkt.k8s.local --region us-east-1 --create-bucket-configuration LocationConstraint=us-east-1

```

the enable the versioning

```
aws s3api put-bucket-versioning --bucket sunil-kops-testbkt.k8s.local --region us-east-1 --versioning-configuration Status=Enabled

```

Exporting the arthifact/data

```
export KOPS_STATE_STORE=s3://sunil-kops-testbkt.k8s.local
```

Creating the cluster

```
kops create cluster --name sunil.k8s.local --zones us-east-1a --master-count=1 --master-size t2.medium --node-count=2 --node-size t2.micro
```

```
kops update cluster --name sunil.k8s.local --yes --admin
```

to check if the cluseter is ready to use 
```
kops validate cluster --wait 10m
```

verify the cluster with the following command `kubectl get nodes` you should see 1 master node(control plane) and 2 worker nodes

To scale in and scale the worker nodes
```kops edit ig --name=sunil.k8s.local master-us-east-1a``` and change the min size and max size

then `kops update cluster --name sunil.k8s.local --yes --admin`

and `kops rolling-update cluster --yes`

to **delete the cluster** 
```
kops delete cluster --name sunil.k8s.local --yes
```

## Creating Namespaces

To create the namesapce `kubectl create ns dev`

to view in which namespace you are `kubectl config view`

th change to another namespace `kubectl config set-context --current --namespace=dev`


## Creating roles and permissions

***Step-1** Create a Service Account file which stores all the users `vi serviceaccount.yml`

```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: john
  namespace: dev
```
then `kubectl create -f serviceaccount.yml`

**Step-2** Create a role file for the roles `vi role.yml`

```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-read-role
  namespace: dev
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "watch", "list"]
```
then `kubectl create -f role.yml`

**Step-3** Create a role binding file where you attach the role to the user in a specific namespace `vi rolebinding.yml`

```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata: 
  name: read-pods
  namespace: dev
subjects:
  - kind: ServiceAccount
    name: john
    namespace: dev
roleRef:
  kind: Role
  name: pod-read-role
  apiGroup: rbac.authorization.k8s.io
```
then `kubectl create -f rolebinding.yml`

test it with ` kubectl auth can-i list pods -- namespace=dev --as=john `  it should return *yes*
