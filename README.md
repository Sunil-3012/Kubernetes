# Kubernetes

**Topics:**

🔹 **Minikube**

🔹 **replicaSet**

🔹 **Deployment**

🔹 **Creating a Job**

🔹 **Creating a cluster**

🔹 **Creating Namspaces**

🔹 **Creating users,roles and permissions**

🔹 **Exposing the Applications via Services - Cluster IP, NodePort and Load Balancer**

🔹 **Deamon Set**

🔹 **Kubernetes Metric Server (Heapster) aka Autoscaling**

🔹 **Volumes(PV & PVC)**

🔹 **Resource Quota**



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
aws s3api create-bucket --bucket sunil-kops-testbkt.k8s.local --region us-east-1    //// Chnage the bucket name

```

To create a bucket in another regrion other than us-east-1 you need to use

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
kops create cluster --name sunil.k8s.local --zones us-east-1a --master-count=1 --master-size t2.medium --node-count=2 --node-size t2.micro   // change your cluster name
```

```
kops update cluster --name sunil.k8s.local --yes --admin
```

to check if the cluseter is ready to use 
```
kops validate cluster --wait 10m
```

verify the cluster with the following command `kubectl get nodes` you should see 1 master node(control plane) and 2 worker nodes

## To scale in and scale the worker nodes and edit the worker nodes or cluster
```kops edit ig --name=sunil.k8s.local master-us-east-1a``` and change the min size and max size

then `kops update cluster --name sunil.k8s.local --yes --admin`

and `kops rolling-update cluster --yes`

To edit the cluster `kops edit cluster sunil.k8s.local `

To edit the nodes and change the instance type `kops edit ig nodes-us-east-1a --name sunil.k8s.local`

then `kops update cluster --name sunil.k8s.local --yes`

then `kops rolling-update cluster --name sunil.k8s.local --yes`

Now relax for few minutes, K8s will do the job

to **delete the cluster** 
```
kops delete cluster --name sunil.k8s.local --yes
```

## Creating Namespaces

To create the namesapce `kubectl create ns dev`

to view in which namespace you are `kubectl config view`

to change to another namespace `kubectl config set-context --current --namespace=dev`

or to go back to default namspace `kubectl congig use-context minikube`


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

## Exposing the Applications via Services - Cluster IP, NodePort and Load Balancer

### Cluster IP
This is the default type of Service. It exposes the Service on a cluster-internal IP. Other services within the same Kubernetes cluster can access the Service, but it is not accessible from outside the cluster.

This creates a connection using an internal Cluster IP address and a Port.

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

---

apiVersion: v1
kind: Service
metadata:
  name: ib-service
spec:
  type: ClusterIP
  selector:
    app: bank
  ports:
    - port: 80
      targetPort: 80
```

### Node Port

This type of Service exposes the Service on each Node’s IP at a static port. A NodePort Service is accessible from outside the cluster by hitting the <NodeIP>:<NodePort>.

When a NodePort is created, kube-proxy exposes a port in the range 30000-32767:

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

---

apiVersion: v1
kind: Service
metadata:
  name: ib-service
spec:
  type: NodePort
  selector:
    app: bank
  ports:
    - port: 80
      targetPort: 80
      nodePort: 31143  //completely optional. the node port should range in between 30000-32767
```

### Load Balancer
This Service type exposes the Service externally using a cloud provider’s load balancer. It is typically used in cloud environments like AWS, GCP, or Azure.

A LoadBalancer is a Kubernetes service that:

Creates a service like ClusterIP
Opens a port in every node like NodePort
Uses a LoadBalancer implementation from The cloud provider (your cloud provider needs to support this for LoadBalancers to work).


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

---

apiVersion: v1
kind: Service
metadata:
  name: ib-service
spec:
  type: LoadBalancer
  selector:
    app: bank
  ports:
    - port: 80
      targetPort: 80
      nodePort: 31143  
```

**To access the application**

-> Go to Security groups and and select nodes security group and edit the inbound rules with the specific port number or allow all traffic and select anywhere IpV4

-> type in this command `kubectl get svc` and you will get the application link or else go to aws console and load balancer and there you will find the url of the application

## Deamon Set

✔️ DaemonSets ensure one pod per node for system-level services.

✔️ Used for monitoring, logging, networking, and security.

✔️ Automatically adds/removes pods when nodes join/leave the cluster.

```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: ib-daemon
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

## Kubernetes Metric Server (Heapster)

It's a scalable, efficient source for monitoring the overall health and performance of a Kubernetes cluster, providing the data needed for Kubernetes features like Horizontal Pod Autoscaler (HPA) and the Kubernetes Dashboard.

This metric server in K8S will collect metrics information like cpu, ram etc for all pods and nodes in the cluster

A single deployment that works on most clusters , collect metrics every 15 secs **`kubectl top pods/ nodes`**

**Installing Metriic Server**

`kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/high-availability-1.21+.yaml`

Creating a sample deployment 

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

We can autoscale using the following command **`kubectl auotscale deploy ib-deployment --cpu-percent=20 --min=2 --max=10`**

To deploy it using the manifest file

```
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: ib-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: ib-deployment
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 20
```

To check if the autoscaling is working or not we do stress testing on the container

login to the container `kubectl exec -it contID -- /bin/bash`

and run the following command

```
apt update
apt install stress
stress --cpu 8 --io 4 --vm 2 --vm-bytes 128M --timeout 60s
```

Watch the pods autoscale
**`kubectl get pods --watch`**


## Volumes 

### PV - Persistent Volume

there are two type of persistent volumes

1) **Stateless**: if the pod is deleted the data is lost, because data is stored locally on the pod and instance

2) **Stateful**: if the pod is deleted the data is persistent, because we can store the data in external storage like AWS EBS

### PVC - Persistent Volume Claim

TO use PV we need to claim the volume using PVC

PVC request a PV with your desired specification(size, access, modes & speed etc) from K8S and once a suitable PV is found it will bound to PVC

To create a Pv first create a **volume in aws ec2 instance with 10Gi magnetic volume and copy the volume ID**

then create a PV yaml file `vi pv.yml`
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  awsElasticBlockStore:
    volumeID: vol-0a04bb207090810bd // copy the volume ID
    fsType: ext4
```

Then create a PVC `vi pvc.yml`

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec: 
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```
Then we create the deployment file mounting the volume to the container and claiming the PV

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pvdeploy
spec:
  replicas: 1
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
        volumeMounts:
        - name: my-pv
          mountPath: "/tmp/persistent"
      volumes:
        - name: my-pv
          persistentVolumeClaim:
            claimName: my-pvc
```

To check if it's working or not

login intio the container and go to `cd /tmp/persistent` and add few files

Now delete the pod, A new pod will be created automatically and login to the new pod and again go to `cd /tmp/persistent` and you will see your files


## Resource Quota

To give limitation to a namespace for the pods,cpy, memory and various other things

Create a sample namspace as Dev `kubectl create ns dev`

and change the namespace from default to dev `kubectl config set-context --current --namespace=dev`

use the below code to set the limitations where the max pods are 5, cpu=1, memory=1GB

```
apiVersion: v1
kind: ResourceQuota
metadata:
  name: dev-quota
  namespace: dev
spec:
  hard:
    pods: "5"
    limits.cpu: "1"
    limits.memory: 1Gi
```

Use the sample deployment to test it 

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
          resources:
            limits:
              cpu: "0.3"
              memory: 300Mi
```

this will work cause for 1 pod the req cpu is 0.3 and memory is 300 and there will be a total of 3 pods created.

To test with different code

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
          resources:
            limits:
              cpu: "1"
              memory: 512Mi
```

## Ingress (Path Based Routing)

* Ingress helps to expose HTTP and HTTPS routes from outside of the Cluster

* Ingress supports Host based routing and path based routing

* Ingress supports load balancing and SSL termination

* IT redirect the incoming requests to the right services based on the web url or path in the address

* Ingress provides encryption feature and helps to balance the load of the applications

To Download the ingress copy this command **'kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.3.0/deploy/static/provider/cloud/deploy.yaml
'**

you need to create few  different deployments(Path) with clusterIP services and create an ingress file to connect with that deployments based on your requirements. Here Nginx acts as a load balancer

Create few different deployment 

#### Internet banking deployment

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: internet-banking  
spec:
  replicas: 2
  selector:
    matchLabels:
      app: ibbank
  template:
    metadata:
      labels:
        app: ibbank
    spec:
      containers:
      - name: ibcont
        image: sunil3012/ib-image:latest
        ports:
        - containerPort: 80
        env:
        - name: TITLE
          value: "internet-banking"
---
apiVersion: v1
kind: Service
metadata:
  name: internet-banking  
spec:
  type: ClusterIP
  ports:
  - port: 80
  selector:
    app: ibbank
```

#### Mobile banking deployment

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mobile-banking  
spec:
  replicas: 2
  selector:
    matchLabels:
      app: mbbank
  template:
    metadata:
      labels:
        app: mbbank
    spec:
      containers:
      - name: mbcont
        image: sunil3012/mb-image:latest
        ports:
        - containerPort: 80
        env:
        - name: TITLE
          value: "mobile-banking"
---
apiVersion: v1
kind: Service
metadata:
  name: mobile-banking
spec:
  type: ClusterIP
  ports:
  - port: 80
  selector:
    app: mbbank
```

#### Ingress file

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: k8s-ingress
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
    nginx.ingress.kubernetes.io/use-regex: "true"
    nginx.ingress.kubernetes.io/rewrite-target: /$2
spec:
  ingressClassName: nginx
  rules:
    - http:
        paths:
          - path: /mobile-banking(/|$)(.*)
            pathType: ImplementationSpecific
            backend:
              service:
                name: mobile-banking
                port:
                  number: 80
          - path: /internet-banking(/|$)(.*)
            pathType: ImplementationSpecific
            backend:
              service:
                name: internet-banking
                port:
                  number: 80
          - path: /(.*)
            pathType: ImplementationSpecific
            backend:
              service:
                name: internet-banking
                port:
                  number: 80
```
Now create all these deployments. To get the ingress and load balancer url `kubectl get ing`
