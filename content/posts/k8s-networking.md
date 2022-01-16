---
title: "Traffic flow in Kubernetes 101" 
date : "2022-01-16"
---
## Table of contents
1. Intra pod communication
2. Inter pod communication
3. CNI plugin
4. Pod to service communication

## Intra pod communication
When pod is created, 
1. It is assigned to a node.
2. A network namespace is created for it(i.e all of that pod's containers belong to that namespace), 
3. IP is assigned to it. 

**For every pod in the cluster there is pause container running in the background. The pause container creates and holds network namespace for that pod.**

So what are the reasons to have pause containers for pods :
1. Basis for sharing the namespace amongst the pod containers
2. Act as init or PID=1 proccess for pod namespace. 

Detailed discussion on this topic : [almighty-pause-container](https://www.ianlewis.org/en/almighty-pause-container)

Now, the pod and all its containers have same IP. Applications or containers inside a pod essentially communicate with each other as if they are on localhost itself. 

## Interpod communication

Interpod communication can happen between :
1. pods belonging to same node
2. pods belonging to different nodes. 

A *tunnel* is created between node's root namespace and pod's namespace using virtual ethernet(veth) pairs. After these veth pairs are created each *tunnel* will have its end in the root namespace. **And communication between same node pod's happens inside root namespace using these.**

A virtual bridge(or 'switch') act as intermediatory to connect these ethernet pair ends in root namespace to each other.

*But then what about pods that live in different nodes?*

Here there is just an extra step than before. When pod tries to connect to other pod the ip address in not found in local network of that node(as it lies in other node) so this request is sent to default gateway which will route it to the desired node where the receiver pod lives and connection is established. 

## CNI plugin(Container Network Interface)

- CNI plugin satisfies the k8s networking requirements for a node (so each k8s node has to install this plugin). 
- As name suggests it is pluggable and many options are present which follow the CNI standard e.g cillium, flannel, calico etc. 
- Difference between them is that they follow different networking setups and strategies that give them various advantages and disadvantages.
- Without CNI plugins things like creating pod namespaces, created virtual ethernet pairs, bridges, setting up routes, networking would have to be done manually. Also, their updation and deletion will have to be done manually. Which seems like a cumbersome task.

## Pod to service communication

When a service is created it is given a static IP address which doesn't change but pod's IP can change when restarted. Without going in too much details what happens:
1. Pod initiates request to service IP
2. A hook is triggered which does a DNAT change and changes the IP address of service to IP address of destination pod. (This change or connection information stored)
3. When the destination pod sends the response another hook is triggered which does SNAT and source of our destination pod is changed to original service IP (using information stored in step 3).

So this way communication happens between 2 pod's with services in between them.

References :
1. https://learnk8s.io/kubernetes-network-packets
