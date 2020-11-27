# K8S - Scheduling 

#### Manual Scheduling

Scheduler adds nodename in the podspec.. 
Scheduler scans all the available nodes and identifies the right node for the pod 

```
spec:
  containers:
    - name:
   
  nodeName: 

  
```
What if there is no scheduler ?.
Create a pod-binding to assign pod to the node 

```
apiVersion: v1
kind: Binding
metadata:
target:
  apiVersion: v1
  kind: Node
  name: node02
```

```
curl --header "Content-Type: application/json" --request POST --data 
'{apiVersion:v1....}'  http://$Server/api/v1/namespaces/default/pods/$PODNAME/binding
```



#### Label and selectors

```
metadata: 
  name: simple-webapp
  labels:
    app: App1
    function: front-end
  annotations:
    buildversion: 1.34  
```

```
kubectl get pods --selector app=App1
```

In replicaset/deployment, you would see labels in two places, 
1. under metadata (under label)
2. under template (This is for pod)

#### Taints and Tolerations
Taints are set on node
Tolerations are set on pods

```
kubectl taint nodes node-name key=value:taint-effect

kubectl taint nodes node1 app=blue:NoSchedule
```

The taint-effects are 

- NoSchedule
- PreferNoSchedule
- NoExecute


```
pod-defn
-------
spec:
  tolerations:
    - key
```

***Tip***: Taint and pod tolerations does not guarantee that your pod will go the specific node. The scheduler may very well place the pod on any other node.

<b><u> Tip:  </u></b>
```
chekout the taint in the master node
```

#### Node Selectors

To make this work, you need to label the node
```
kubectl label nodes node-1 size=Large
```

```
spec:
  nodeSelector:
    size: Large
```

It has limitations. you cannot provide or / and condition and works on the single label option. 

If you were to place on medium or large then 

#### Node affinity

you can provide more than one condition in the nodeAffinity

```
spec:
  containers:

  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoreDuringException:
        nodeSelectorTerms:
          -matchExpressions:
            - key: size
              operator: In
              values:      
                - Large
                - Medium

```

***Tip***

There are two different attributes in the spec column

```
spec:
  nodeSelector:
```

```
spec:
  nodeName: <>
```

Pod Selector with labels

```
kubectl get pods --selector env=dev
kubectl get all --selector env=prod
kubectl get all --selector env=prod,bu=finance,tier=frontend
```

```

```

#### Taints and Tolerations

Create a taint on a node

on a node key=value with NoSchedule
```
kubectl taint nodes node01 spray=mortein:NoSchedule 
```

(To add taint) 
[https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/]


```
tolerations:
- key: "key"
  operator: "Equal"
  value: "value"
  effect: "NoSchedule"
```

***Tip***
Some
```
kubectl get nodes -o json | jq '.items[].spec.taints'

o/p:
[
  {
    "effect": "NoSchedule",
    "key": "node-role.kubernetes.io/master"
  }
]
[
  {
    "effect": "NoSchedule",
    "key": "spray",
    "value": "mortein"
  }
]


kubectl taint nodes controlplane node-role.kubernetes.io/master:NoSchedule-

```


#### Label

Add a label

```
kubectl label node node01 color=blue
kubectl create deployment blue --image=nginx --replicas=6 --dry-run=client -o yaml > nginx6.yaml
```

```
kubectl create deployment red --image=nginx --replicas=3 --dry-run=client -o yaml > red1.yaml
```

Refer blue.yaml and red.yaml

<b><u> Tip </u></b>
Taints and Tolerations and Node affinity must be used in combination to make this happen. 

#### Resource Limits

```
https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/memory-default-namespace/
```

```
apiVersion: v1
kind: LimitRange
metadata:
    name: mem-limit-range
spec:
    limits:
    - default:
        memory: 512Mi
    defaultRequest:
        memory: 256Mi
    type: Container
```

***Tip*** 
Refer doc - k8s io


#### Daemon Sets

Kube-proxy

networking

There is no way for you create the daemonset imperatively, 
so you need to use 
`kubectl create deployment -o yaml`
and then modify the following fields in the yaml file
`replicas, creationTimestamp (in meta) and Deployment -> DaemonSet `


#### Static PODs

This is about kubelet architecture & flow

Where is the kubelet options?,, 

Kubelet will read from `/etc/kubernetes/manifests` and creates the pod.. 
Follow the kubelet service or config .. 

you can only create PODs not any other objects

Look for the attribute called `--pod-manifest`

Cluster set up by the kubeadm tool uses

kubelet.service
`config=kubeconfig.yaml`


Static mirror path <br>
Any pod that is created through the static pods (kubelet), it creates the mirror pod in the kube-api server. you cannot edit or do anything with that pod. 
Not dependent on the control plane..

<br>





Side Note:

In Kubeadm setup 

```
root@kind-control-plane:/etc/kubernetes/manifests# ls -ltr
total 16
-rw------- 1 root root 3692 Nov 22 19:28 kube-apiserver.yaml
-rw------- 1 root root 3380 Nov 22 19:28 kube-controller-manager.yaml
-rw------- 1 root root 1384 Nov 22 19:28 kube-scheduler.yaml
-rw------- 1 root root 2115 Nov 22 19:28 etcd.yaml
root@kind-control-plane:/etc/kubernetes/manifests#
```

First, look for `staticPodPath` or look for `config` option. 

kubelet service

```
oot@kind-control-plane:/etc/systemd/system# cat kubelet.service
# slightly modified from:
# https://github.com/kubernetes/kubernetes/blob/ba8fcafaf8c502a454acd86b728c857932555315/build/debs/kubelet.service
[Unit]
Description=kubelet: The Kubernetes Node Agent
Documentation=http://kubernetes.io/docs/

[Service]
ExecStart=/usr/bin/kubelet
Restart=always
StartLimitInterval=0
# NOTE: kind deviates from upstream here with a lower RestartSecuse
RestartSec=1s

[Install]
WantedBy=multi-user.target
```

***Important Tips Questions***

How many static pods are running?
```
❯ kubectl get pods --all-namespaces | grep contr
calico-system     calico-kube-controllers-85ff5cb957-qdd7q     1/1     Running   0          2d23h
kube-system       etcd-kind-control-plane                      1/1     Running   0          2d23h
kube-system       kube-apiserver-kind-control-plane            1/1     Running   0          2d23h
kube-system       kube-controller-manager-kind-control-plane   1/1     Running   10         2d23h
kube-system       kube-scheduler-kind-control-plane            1/1     Running   12         2d23h
```

***DNS and kube-proxy are not part of the control plane***

***The static pods are created only on master (control plane)***

What is the path of the directory?
<br>
First look for where is kubelet running


Run the command `ps -aux | grep kubelet` and identify the config `file - --config=/var/lib/kubelet/config.yaml`. Then checkin the config file for `staticPodPath`.


In kind setup -> with kubeadm path
```
/usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --container-runtime=remote --container-runtime-endpoint=unix:///run/containerd/containerd.sock --fail-swap-on=false --node-ip=192.168.48.5 --provider-id=kind://docker/kind/kind-control-plane --fail-swap-on=false
```
/var/lib/kubelet/config.yaml
```
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
    enabled: false
  webhook:
    cacheTTL: 0s
    enabled: true
  x509:
    clientCAFile: /etc/kubernetes/pki/ca.crt
authorization:
  mode: Webhook
  webhook:
    cacheAuthorizedTTL: 0s
    cacheUnauthorizedTTL: 0s
clusterDNS:
- 10.96.0.10
clusterDomain: cluster.local
cpuManagerReconcilePeriod: 0s
evictionHard:
  imagefs.available: 0%
  nodefs.available: 0%
  nodefs.inodesFree: 0%
evictionPressureTransitionPeriod: 0s
fileCheckFrequency: 0s
healthzBindAddress: 127.0.0.1
healthzPort: 10248
httpCheckFrequency: 0s
imageGCHighThresholdPercent: 100
imageMinimumGCAge: 0s
kind: KubeletConfiguration
logging: {}
nodeStatusReportFrequency: 0s
nodeStatusUpdateFrequency: 0s
rotateCertificates: true
runtimeRequestTimeout: 0s
staticPodPath: /etc/kubernetes/manifests
streamingConnectionIdleTimeout: 0s
syncFrequency: 0s
volumeStatsAggPeriod: 0s
```

To find out where is the kubelet

```
ps -ef | grep kubelet | grep "\--config"
```


Create a static pod
```
kubectl run static-busybox --image=busybox --dry-run=client -o yaml --command -- sleep 1000 > /etc/kubernetes/manifests/static-pod.yaml

Simply edit the static pod definition file and save it. If that does not re-create the pod, run: 

'kubectl run --restart=Never --image=busybox:1.28.4 static-busybox --dry-run=client -o yaml --command -- sleep 1000 > /etc/kubernetes/manifests/static-busybox.yaml'

```


#### Multiple Schedulers

You can have your own scheduler for your needs .. 
Mutiple scheduler can be added. 

kube-scheduler.service

deploy your own scheduler 

find out the kube-scheduler.. and play with that

The key difference is that 

```
pod-spec:
--------
spec:
  containers:
    - command:
        - kube-scheduler
        - ....
        - --scheduler-name=my-scheduler
        - --leader-elect=true

```

When you have multiple copies of the same scheduler running in HA setup , you need to have only one active in the system. so, set the system to run as `--leader-elect=true`

Say, when you have multiple schedulers are set up in your system, you need to add `- --lock-object-name=my-scheduler`, so that it wont get participated in the leader elect process. 


In pod-spec ->
```
spec:
  containers:


  schedulerName: your-scheduler-name
```

To understand which scheduler picked it up  
```
kubectl get events
kubectl logs my-scheduler --namespace=kube-system
```

???
pod name, container name in a pod spec => spend a cycle ??



