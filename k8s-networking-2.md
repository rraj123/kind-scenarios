#### Explore networking 


kube-scheduler mode 
```
netstat -nplt
netstat -anp | grep etcd
```

How do you find out the networking plugin used 
```
ps -aux | grep kubelet

look for `--network-plugin=cni`
look for --cni-bin-dir  `--cni-bin-dir=/opt/cni/bin`

```
network plugin
```
ls /etc/cni/net.d/
```

#### Deploy networking 
```
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```

#### Networking Weave

```
/etc/cni/net.d/
```
ip link will show you the network interface 
```
ip link
ip addr show <<weave>> (show you the IP Range)
```

#### Service Networking 
```
kubectl -n kube-system logs weave-net-7txk8 -c weave | grep ipalloc
```

ip range
```
cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep cluster-ip-range
ans:
    - --service-cluster-ip-range=10.96.0.0/12
```

There are different modes in the ipTables.. 

#### Core DNS 

```
kubectl -n kube-system describe deployments.apps coredns | grep -A2 Args | grep Corefile

```

Here is the output
```
 kubectl exec -it hr -- nslookup mysql.payroll > /root/CKA/nslookup.out
```