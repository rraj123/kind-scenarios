### K8s - Application Failure Scenarios.

What are all the things that could go wrong?. 

Services: <br>
+ Check the Service Name, selector, Port number 
+ Does services correctly expose the pod / port number?
+ Does Service correctly selects the pod (Lables) - 



First check the service, if they have the endpoint
`kubectl get ep`

--


### Worker Failure 

First thing, read more on the `troubleshooting clusters` section in the kubernetes.io site. 

Overall, This breaks down the concepts very well. 

First Thing, take a look at the `kubelet`

```
systemctl status kubelet.service -l
```

If the kubelet is not active, The stop would be to take a look at the 

```
journalctl -u kubelet
```
sft+g to navigate to the different 

The next step is to take a look at the kubelet startup script

```
cd /etc/systemd/system/kubelet.service.d/
ls 

```

Look at the kubeadm.conf

```
open 
cat 10-kubeadm.conf

look for 
ENVIRONEMNT

KUBELET_KUBECONFIG_ARGS=--bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf

KUBELET_CONFIG_ARGS=--config=/var/lib/kubelet/config.yaml

```

Look at the 
```
cat /var/lib/kubelet/config.yaml
```

you can view the 

file here .. 

The kublet configuration is similar to any other k8s configuration.. 

```

```

Once you make the change then you need to reload the service 

```
systemctl daemon-reload
systemctl restart kubelet
```

#### Kubelet Error

First you need to check
1. status
2. journalctl -u 

look at the error


`kubectl cluster-info`

*** understand what is in kubeconfig and kublet

```
cd /etc/systemd/system/kubelet.service.d/
```



### Network 
<br>


Kube proxy is slightly different from other components. So, debugging this  would be slightly different. 

`kubectl -n kube-system get events` will not get you anything. 

`kubectl -n kube-system get pods`

Lets say that if you have proxy is not running, then you need to take a look at the configuration. 

Here, the kube-proxy is triggered in the container, and the configuration is mounted as config volume. So, check the config volume also. 


So, check the config map in the kubsystem namespace

`kubectl -n kube-system get cm`

So, look at the details
```
ontrolplane $ kubectl -n kube-system describe cm kube-proxy
Name:         kube-proxy
Namespace:    kube-system
Labels:       app=kube-proxy
Annotations:  kubeadm.kubernetes.io/component-config.hash: sha256:582cce4e8806c2a4a77bddc32086dded16dbd38c628c720eb758213d59027c5d

Data
====
config.conf:
----
apiVersion: kubeproxy.config.k8s.io/v1alpha1
bindAddress: 0.0.0.0
bindAddressHardFail: false
clientConnection:
  acceptContentTypes: ""
  burst: 0
  contentType: ""
  kubeconfig: /var/lib/kube-proxy/kubeconfig.conf
  qps: 0
clusterCIDR: 10.244.0.0/16
configSyncPeriod: 0s
conntrack:
  maxPerCore: null
  min: null
  tcpCloseWaitTimeout: null
  tcpEstablishedTimeout: null
detectLocalMode: ""
enableProfiling: false
healthzBindAddress: ""
hostnameOverride: ""
iptables:
  masqueradeAll: false
  masqueradeBit: null
  minSyncPeriod: 0s
  syncPeriod: 0s
ipvs:
  excludeCIDRs: null
  minSyncPeriod: 0s
```


