# Taints and Tolerations (Imporatant for Interviews)

What is kube-scheduler? 
Kube-scheduler is responsible to schedule pods on to woker-nodes. Kube-scheduler is master node component 

- whenever we type kubectl apply -f manifest.yaml, first it checks for authentication 
that if you have any access to apply. 
- It scans for resources and checks if you have access or not that is decided by the kubernetes. It confirms that if you have access and then it hand over to kube-scheduler 

-> kube-scheduler checks for worker-nodes and whichever worker-node is free and it allocates the randome one 

eg:
nodeSelector:
	az: us-east-1b 
	
taint --> paint 

Example: 
- Banks and RBI may accept painted notes -> It means they can tolerate (tolerate in the sense excuse)
- We can taint the node(it means it is polluted). For suppose if you tainted any node, then kube-scheduler will not schedule any pod inside that node 
- kube-scheduler cannot schedule any pod in that node 
- GPU based servers are required for expense project 
- taint these GPU nodes 
- expense project users should give toleration in their manifest files 
- tolerations are written by users in expense project 
- manily taints and tolerations means rejecting the other project related pods are not allowed to nodes 

- Taints are opposite -> they allow a node to repel (reject) a set of pods 
- Tolerations are applied to pods. Tolerations allow the scheduler to schedule pods with matching taints 
- Tolerations allow scheduling but don't guarantee scheduling

nodeSelector:
	label-key: value
	
taints and tolerations --> to repel the pods. We can mark the node as tainted so that scheduler will not schedule any pods on to it 

If you apply tolerate, scheduler can schedule the pods on to tainted nodes, but not guaranted.

**Use-Cases of Taint & Tolerations**
1. Project specific worker nodes 
2. Special hardware. GPU based servers 

Now let us apply the concept.
- List the nodes in the cluster
```
kubectl get nodes
```
- Command to taint the node
```
kubectl taint nodes ip-192-168-2-237.ec2.internal hardware=gpu:NoSchedule
```
- If we run the above command, then no pod is scheduled to this node.

Now, lets tolerate a pod

**01-tolerations.yaml**

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  # docker run -d --name nginx nginx
  - name: nginx
    image: nginx:stable-perl
```

Now, apply the below commands

```
kubectl apply -f 01-tolerations.yaml
```
```
kubectl get pods -o wide
```
- Here, we can see that pod is not there in the tainted node
- We can also describe the tainted node to see that the node is tainted using the below command
``` 
kubectl describe node ip-192-168-2-237.ec2.internal
```
- Now, to send the pod to the tainted node, we will use tolerations. So, we will delete the pod
```
kubectl delete -f 01-tolerations.yaml
```

**01-tolerations.yaml**

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  # docker run -d --name nginx nginx
  - name: nginx
    image: nginx:stable-perl
  tolerations:
  - key: "hardware"
    operator: "Equal"
    value: "gpu"
    effect: "NoSchedule"
```

- Apply the below commands. The pod may or maynot goes to the tainted node. It is not gaurentee

```
kubectl apply -f 01-tolerations.yaml
```
```
kubectl get pods -o wide 
```

- If we want the POD to go the tainted node (gaurentee), then we will use affinity and anti-affinity

In tolerations, there is `effect` field with multiple options. 
1. NoSchedule: It won't schedule the new pods
2. NoExecute: Existing Pods won't be scheduled. Existing pods will go and run on another workernode  

**Summary of the Key Differences:**
- **NoSchedule:**
	- Only new Pods are affected.
	- Existing Pods remain unaffected and continue running.
- **NoExecute:**
	- Both new Pods and existing Pods are affected.
	- Existing Pods without the toleration are evicted from the node.


**Adding Label and nodeSelector**

Add a Label to a Node

```
kubectl label nodes ip-192-168-2-237.ec2.internal hardware=gpu
```

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  # docker run -d --name nginx nginx
  - name: nginx
    image: nginx:stable-perl
  nodeSelector:
    hardware: gpu
  tolerations:
  - key: "hardware"
    operator: "Equal"
    value: "gpu"
    effect: "NoSchedule"
```
- Here, we have added nodeSelector

- Apply the below commands. Now, the pod gets into tainted node

```
kubectl apply -f 01-tolerations.yaml
```
```
kubectl get pods -o wide 
```


# Affinity & Anti-affinity

Affinity is nothing but attraction.

There are two types:
1. Node Affinity
2. POD Affinity

Now, lets see about Node Affinity

**02-affinity.yaml**

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: hardware
            operator: In
            values:
            - gpu
  containers:
  - name: nginx
    image: nginx:stable-perl
  tolerations:
  - key: "hardware"
    operator: "Equal"
    value: "gpu"
    effect: "NoSchedule"
```

Apply the below commands:
```
kubectl apply -f 02-affinity.yaml 
```
```
kubectl get pods -o wide
```
- Here, we can see that the pod is in tainted node
- Suppose, if the pod status is pending, we have to notice that node has changed since it is spot instance.
- So, again taint any one of the node and apply the label to that node 

**03-affinity.yaml**

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  affinity:
    nodeAffinity: 
		preferredDuringSchedulingIgnoredDuringExecution:
        - weight: 1
          preference:
            matchExpressions:
			- key: another-node-label-key
			  operator: In
			  values:
			  - another-node-label-value
  containers:
  - name: nginx
    image: nginx:stable-perl
  tolerations:
  - key: "hardware"
    operator: "Equal"
    value: "gpu"
    effect: "NoSchedule"
```

- Here, we can see that still the pod is successfully running even rule is not matched

- Here, in nodeAffinity there are two types of fields: 
**1. requiredDuringSchedulingIgnoredDuringExecution:**
	- The Pod must meet the rule to be scheduled, but once it's running, the rule doesn't matter anymore.
	- This is hard rule
**2. preferredDuringSchedulingIgnoredDuringExecution:**
	- The Pod prefers to meet the rule for scheduling, but it can still be scheduled even if the rule isn't met, and the rule doesn't affect it once it's running.
	- This is soft rule

- The `weight` field in NodeAffinity (under preferredDuringSchedulingIgnoredDuringExecution) tells Kubernetes how strongly to prefer a node for scheduling, with higher numbers meaning stronger preference.


Now, lets see about POD affinity

- Pod Affinity means one pod likes another pod

- Let us create one pod

**04-pod-affinity**

```
apiVersion: v1
kind: Pod
metadata:
  name: pod-1
spec:
  containers:
  - name: nginx
    image: nginx:stable-perl
```

```
kubectl apply -f 04-pod-affinity.yaml
```
```
kubectl get pods -o wide
```

- Now, create another pod with podaffinity and this pod should be created in the same node where pod-1 is created

```
apiVersion: v1
kind: Pod
metadata:
  name: pod-1
spec:
  containers:
  - name: nginx
    image: nginx:stable-perl
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-2
  labels:
	purpose: affinity
spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: purpose
            operator: In
            values:
            - affinity
        topologyKey: topology.kubernetes.io/zone
  containers:
  - name: nginx
    image: nginx:stable-perl
```

```
kubectl apply -f 04-pod-affinity.yaml
```
```
kubectl get pods -o wide
```

- Now, we can see that pod-1 & pod-2 are in same node

**pod anti-affinity**
It goes other than what you have given 

```
apiVersion: v1
kind: Pod
metadata:
  name: pod-1
  labels:
    purpose: affinity
spec:
  containers:
  - name: nginx
    image: nginx:stable-perl
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-2
spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: purpose
            operator: In
            values:
            - affinity
        topologyKey: topology.kubernetes.io/zone
  containers:
  - name: nginx
    image: nginx:stable-perl
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-3
spec:
  affinity:
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: purpose
            operator: In
            values:
            - affinity
        topologyKey: topology.kubernetes.io/zone
  containers:
  - name: nginx
    image: nginx:stable-perl
```
```
kubectl apply -f 04-pod-affinity.yaml
```
```
kubectl get pods -o wide
```

- Here, we can see that pod-1 & pod-2 are in same node and pod-3 is in different node.


**UseCase of Affinity and Anti-affinity**

- Everytime Hitting DataBase is an expensive process  
- For suppose, we have products list and everytime it may doesn't change. It's takes time to get the product details everytime because first we need to connect to DB then run query and fetch results and need to close. This may lead to increase in latency  
- Generally, everyone keeps the cache. So that everytime we cannot hit the DB
- First, backend gets the request from DB and keep it in a cache and if someone request again then it checks in cache and then results goes fast. By this, we can reduce the latency 

![alt text](img/affinity_1.drawio.svg)

- For suppose, backend pod is in one node and cache pod is in another node. Then traffic comes from backend pod and then node of the backend pod and enters into cache's node and last it enters into cache pod 

![alt text](img/affinity_2.drawio.svg)

- This may increases the latency 

- If the backend pod and cache pod are in same node, then we can reduce the latency

![alt text](img/affinity_3.drawio.svg)

- First we will run cache pod and we will inform backend pod to run where cache pod is runnig, then latency get reduced. So, this is called as pod-affinity 

let us take 2 replicas(i.e two backend pods and two cache's)
	- If cache is running in one node and what if we want that cache run at different place, then that is called as anti-affinity 

![alt text](img/antiaffinity.drawio.svg)

- Inter-pod affinity and anti-affinity can be even more useful when tehy are used with higher level collections such as RelicaSets, StatefulSets, Deployments, etc. 

**05-use-case.yaml**

Suppose, if we taint any of the node then it won't allow. So, in such cases we need to untaint using the command
```
kubectl taint nodes ip-192-168-44-187.ec2.internal hardware=gpu:NoSchedule-
```

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-cache
spec:
  selector:
    matchLabels:
      app: store
  replicas: 3
  template:
    metadata:
      labels:
        app: store
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - store
            topologyKey: "kubernetes.io/hostname"
      containers:
      - name: redis-server
        image: redis:3.2-alpine
```

```
kubectl apply -f 05-use-case.yaml
```
```
kubectl get pods -o wide 
```

- Here, we can see that three redis-cache pods are created in three different nodes

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-cache
spec:
  selector:
    matchLabels:
      app: store
  replicas: 3
  template:
    metadata:
      labels:
        app: store
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - store
            topologyKey: "kubernetes.io/hostname"
      containers:
      - name: redis-server
        image: redis:3.2-alpine
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-server
spec:
  selector:
    matchLabels:
      app: web-store
  replicas: 3
  template:
    metadata:
      labels:
        app: web-store
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - web-store
            topologyKey: "kubernetes.io/hostname"
        podAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - store
            topologyKey: "kubernetes.io/hostname"
      containers:
      - name: web-app
        image: nginx:1.16-alpine
```

```
kubectl apply -f 05-use-case.yaml
```
```
kubectl get pods -o wide 
```

- Here also three web-server pods are created in three different nodes

**what is called pod-affinity? (important for interviews)**
- pod-affinity is second pod runs where the first pod is running. second pod likes the first pod 
- we can match labels of first pod into the second pod . so that we make sure second pod is running where the first pod is running 
- Anti-affinity is opposite, second pod will not run where the first pod is running 
