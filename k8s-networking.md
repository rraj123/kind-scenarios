# K8s Networking

### Introduction
1. Switch
2. Routing (connects two n/w,  )
3. Gateway (door to other ) <br>
  `route`  
  
  Add a route 

  `ip route add 192.168.2.0/24 via 192.168.1.1`

  Set up linux host as a router


By default, the packet forward is not enabled .. The setup is done at `cat /proc/sys/net/ipv4/ip_forward`
The value must be set to 1. 
The value must be set in `/etc/sysctl.conf` -> 

```
...
net.ipv4.ip_forward = 1
``` 

Commands:
```
ip link
ip addr
ip addr add `192.168.1.10/24` dev eth0
ip route

```
### DNS - Review

The order of the lookup can be set in 
`cat /etc/nsswitch.conf`


### Network Namespaces

```
ip netns add red
ip netns add blue

ip -n red link
```


#### Practice
```
How do you find out the network interfaces or node to node comm?
ifconfig -a
cat /etc/nework/interfaces
ip link
---
ifconfig 
-- 
ip link show ens3

arp node01 
ip link show docker0 
What is the IP address of the Default Gateway?
ip route show default
```

### Pod Networking

Investigate the kubelet plugins, look at the `--network-plugin=cni`

```
ps -aux | grep kubelet
....
....
--cni-bin-dir 
--cni-conf-dir
--network-plugin


---------

ls /opt/cni/bin

ls /etc/cni/net.d

---
view logs

kubectl logs weave-net-podname weave -n kube-system

```

??? Where do you see the k8s network plugin ???

```
ls /etc/cni/net.d/
cat /etc/cni/net.d/10-weave.conflist
```

plug-in type 

```
$ cat /etc/cni/net.d/10-weave.conflist
{
    "cniVersion": "0.3.0",
    "name": "weave",
    "plugins": [
        {
            "name": "weave",
            "type": "weave-net",
            "hairpinMode": true
        },
        {
            "type": "portmap",
            "capabilities": {"portMappings": true},
            "snat": true
        }
    ]
}
```

#### IP Address Management
CNI 
```
ip addr show weave
```

???
`kubectl run busybox --image=busybox --command sleep 1000 -o yaml --dry-run=client > pod.yaml`

`ip r` or `ip route` to findout the gateway.. 


### Service Networking


kube-proxy is responsible for setting the proxy mode 
1. userspace
2. iptables.
3. ipvs


find out the where is the kube proxy log is .. 
```
kubelet get service (Whatever the service name is feed it in)
iptables -L -t nat | grep db-service 
cat /var/log/kube-proxy.log
```

What is the range of IP addresses configured for PODs on this cluster?

kubectl logs weave-net-7dcwh weave -n kube-system
look for ipalloc-range

What is the IP Range configured for this cluster?
```
cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep cluster-ip-range
```


#### DNS in k8s


```
cat /etc/coredns/Corefile

```
pod has these information.. Who modifies the info on the pods?.. Kubelet. In a nutshell, the pods are configured with the right name server. 
`cat /etc/resolv.conf ` (Watch for this .. )
What happens if there is a core DNS fails. ?. 

Investigate the following file  
`cat /var/lib/kubelet/config.yaml`


### Ingress 

Proxy - b/w DNS and cluster 

Built -in Layer 7

Ingress Controller
   + Must pass the `args : -nginx-ingress-controller`
   + pass the settings info through config map. 
   + Pass pod_name 
   ....
   + additionally, you need to create service nginx-ingress
   + Service Account (View the picture ... (Link))

Ingress Resources
```
apiVersion: extension/v1beta1
kind: Ingress
metadata: 
  name: ingress-wear
spec:
  backend:
    serviceName: hello-service
    servicePort: 80
---
---with  Rules
apiVersion: extension/v1beta1
kind: Ingress
metadata: 
  name: ingress-wear
spec:
  rules:
  - http:
      paths:
      - path: /wear
          backend:
            serviceName: hello-service
            servicePort: 80


--- 
--with domain name 
apiVersion: extension/v1beta1
kind: Ingress
metadata: 
  name: ingress-wear
spec:
  rules:
  - host: wear.mystore.com
    http:
      paths:
      - path: /wear
          backend:
            serviceName: hello-service
            servicePort: 80
---
kubectl apply -f 

kubectl get ingress --> should display the resources. 

kubectl describe ingress <ingress-name>


```

**Practice Tips**
https://kubernetes.github.io/ingress-nginx/examples/rewrite/
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: test-ingress
  namespace: critical-space
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - path: /pay
        backend:
          serviceName: pay-service
          servicePort: 8282
```
***Tip***
```
kubectl -n ingress-space expose deployment ingress-controller --name ingress --port 80 --target-port 80 --type NodePort --dry-run=client -o yaml > ingress-service.yaml
```

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-wear-watch
  namespace: app-space
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
  rules:
  - http:
      paths:
      - path: /wear
        backend:
          serviceName: wear-service
          servicePort: 8080
      - path: /watch
        backend:
          serviceName: video-service
          servicePort: 8080
```