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