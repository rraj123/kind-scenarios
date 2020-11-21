# K8s-Authorization

Entitlement logic, What can i do when i get an access. 

## Mechanisms

1. Node
2. ABAC
3. RBAC
4. Webhook

### Node Authorization

kubelet -> kube API server
    (sends information about node status and etc)

<br>

Kubelet reports to KUBE API for 
   1. pass the info about the node. 
   2. Read (Services, endpoints, Nodes, Pods) / Write (Node status, pod status , events)

<br>

kubelet should be part of "system node group"

Kube API has special authorizer caller "Node authorizer"

<b> Place cert here </b>


### ABAC

User / Group can be associated to <b>a set of permission</b>. 
Create a policy permission file and apply.

Every change requires Kube API server to be restarted. (This would be difficult to manage). 

#### Role based Access System

