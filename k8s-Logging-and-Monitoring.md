# K8s Logging and Monitoring

### Monitoring Components

1. Node level
2. Pod level

Example:
Heapster (Deprecated), Metrics Server (One ), Data dog, Elastic stack, dynatrace 


<b> Metrics Server </b> </br>
1. One per Cluster
2. In Memory
3. Kubelet (cadvisor)

Separate deployment (add on)

`kubectl create -f deploy/1.8+/`

`kubectl top node`
`kubectl top pod`


#### Manage Application logs


`kubectl logs -f pod-name container-name`

```
https://github.com/kodekloudhub/kubernetes-metrics-server.git

kubectl top node
```
