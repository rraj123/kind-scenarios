# kind-scenarios

Introduction:

The objective of this repo is to create a local kind cluster with kubeadm script. Thanks to mauilion

(This is Surya)
## Install kind cluster locally

```
curl -LO git.io/kind-mn-nocni.yaml
```
The script looks like below and disabled the Default CNI

```
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
networking:
  disableDefaultCNI: true
nodes:
- role: control-plane
- role: worker
- role: worker
- role: worker
```

The below is the command to create the cluster
```
kind create cluster --config  kind-mn-nocni.yaml
```
The o/p should look like 

```
Creating cluster "kind" ...
 ✓ Ensuring node image (kindest/node:v1.19.1) 🖼 
 ✓ Preparing nodes 📦 📦 📦 📦  
 ✓ Writing configuration 📜 
 ✓ Starting control-plane 🕹️ 
 ✓ Installing StorageClass 💾 
 ✓ Joining worker nodes 🚜 
Set kubectl context to "kind-kind"
You can now use your cluster with:

kubectl cluster-info --context kind-kind

Have a nice day! 👋
```

The following command will give you an output similar to 

```
❯ kubectl cluster-info --context kind-kind
Kubernetes master is running at https://127.0.0.1:50269
KubeDNS is running at https://127.0.0.1:50269/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```
kubectl will give you following output 

```
❯ kubectl get nodes
NAME                 STATUS     ROLES    AGE   VERSION
kind-control-plane   NotReady   master   36m   v1.19.1
kind-worker          NotReady   <none>   36m   v1.19.1
kind-worker2         NotReady   <none>   36m   v1.19.1
kind-worker3         NotReady   <none>   36m   v1.19.1
```

From docker standpoint, the underlying docker ps will look like 
```
❯ docker ps
CONTAINER ID        IMAGE                  COMMAND                  CREATED             STATUS              PORTS                       NAMES
ec91a6967b68        kindest/node:v1.19.1   "/usr/local/bin/entr…"   44 minutes ago      Up 44 minutes                                   kind-worker3
00af4595aaa2        kindest/node:v1.19.1   "/usr/local/bin/entr…"   44 minutes ago      Up 44 minutes                                   kind-worker2
96d8d01b5ef4        kindest/node:v1.19.1   "/usr/local/bin/entr…"   44 minutes ago      Up 44 minutes       127.0.0.1:50269->6443/tcp   kind-control-plane
622218a08d63        kindest/node:v1.19.1   "/usr/local/bin/entr…"   44 minutes ago      Up 44 minutes                                   kind-worker
e334f910be6a        registry:2             "/entrypoint.sh /etc…"   2 weeks ago         Up 4 days           0.0.0.0:5000->5000/tcp      registry
```

Try to bash into one of the control plane node and explore .. 

```
docker exec -it kind-control-plane bash
```
The control plane node should be running all the core components. 

```
root@kind-control-plane:/# crictl ps
```
Each of the node has containerd running .. 
```


CONTAINER           IMAGE               CREATED             STATE               NAME                      ATTEMPT             POD ID
6b84037682223       47e289e332426       48 minutes ago      Running             kube-proxy                0                   00ba57efe8977
84e5abb2920c7       0369cf4303ffd       49 minutes ago      Running             etcd                      0                   0fe7d4dcab653
24bf8c4fde84f       8cba89a89aaa8       49 minutes ago      Running             kube-apiserver            0                   ad86b88dcc3eb
1b65defdb3ad2       7dafbafe72c90       49 minutes ago      Running             kube-controller-manager   0                   02ddc94f5db77
0a83860c5e59a       4d648fc900179       49 minutes ago      Running             kube-scheduler            0                   27f58107f2ddf
```

### Reset the kind cluster manually. [ kubeadm reset ]
The intent is to clean out all the core components of the  control plane and worker nodes. Now, bash into the control plane and use kubeadm reset to wipe out all of the core components. 

```
docker exec -it kind-control-plane kubeadm reset -f
docker exec -it kind-worker kubeadm reset -f
docker exec -it kind-worker2 kubeadm reset -f
docker exec -it kind-worker3 kubeadm reset -f

```

Now, kubectl will not work, because the previous command deleted all the core components. try that 

```
kubectl get nodes [or get pods]
```

Try bash into the control plane node and explore the pods. 
```
docker exec -it kind-control-plane  bash
```

The crictl ps will not give you anything.. 

```
root@kind-control-plane:/# crictl ps
CONTAINER           IMAGE               CREATED             STATE               NAME                ATTEMPT             POD ID
root@kind-control-plane:/# 
```

Currently, we have four nodes with nothing. 

```
docker ps
```
will list all the four containers (nodes)

Now, bash into the control plane node 

```
docker exec -it kind-control-plane  bash
```
kind has kubeadm configuration in the node itself. 

```
cat /kind/kubeadm.conf
```
Take a moment and explore all the section in the kubeadm configuration. 

Here is the snippet of kubeadm configuration. 
```
apiServer:
  certSANs:
  - localhost
  - 127.0.0.1
  extraArgs:
    runtime-config: ""
apiVersion: kubeadm.k8s.io/v1beta2
clusterName: kind
controlPlaneEndpoint: kind-control-plane:6443
controllerManager:
  extraArgs:
    enable-hostpath-provisioner: "true"
kind: ClusterConfiguration
kubernetesVersion: v1.19.1
networking:
  podSubnet: 10.244.0.0/16
  serviceSubnet: 10.96.0.0/16
scheduler:
  extraArgs: null

```
Pay attention to the controlPlaneeEndpoint (uses docker dns for routing)
Enable-host-path-provisioner: (The static pods uses this initialize, Try to connect the dots.. to run the staeful pods and etc)
podSubnet
serviceSubnet


Go through the 
k8s version
Init configuration
Join configuration
kubelet conf
kubeproxy configuration.

### Configure control plane node using kubeadm 
bash into control plane container

```
docker exec -it kind-control-plane bash
```

? swap enabled --
? privileged container

```
kubeadm init --config=/kind/kubeadm.conf --ignore-preflight-errors=all
```
Go through the kubeadm output ... 

Try kubectl get nodes, nothing would work. 

Set up KUBECONFIG 

```
export KUBECONFIG=/etc/kubernetes/admin.conf
kubectl get nodes
```

Now, you have a control plane node that has all the kubernetes components installed. 

The master status is set to NotReady, Because, it does not have any CNI installed. 

The next step is to install the CNI. 

### Install Calico (CNI)
Inside the control plane container. 

If you are not 
```
docker exec -it kind-control-plane bash
curl -LO git.io/calico.sh
cat calico.sh
```
Go through the calico.sh file

Install the calico operator file
```
kubectl apply -f https://docs.projectcalico.org/manifests/tigera-operator.yaml
```

The above kubectl does not apply calico pod network. 

```
curl -LO https://docs.projectcalico.org/manifests/custom-resources.yaml

cat custom-resources.yaml
```
you should see that CIDR range is very different from the kubeadm.conf file. 

```
grep -i pod /kind/kubeadm.conf
```

you should see different podsubnet 

replace the cidr range in custom-resources  
```
sed -i s/192.168.0.0/10.244.0.0/ custom-resources.yaml
```

```
kubectl apply -f custom-resources.yaml
```

To find out the status ..

```
kubectl get pods -Aw
```

After a couple of seconds, you should have one node cluster that has everything installed. 


```
kubectl get pods
```
Now check the node status

```
kubectl get nodes
```
you should see all status of the master node...

### Grow the cluster

To get the kubeadm token from the master node, 

Bash into the control plane node

```
kubeadm token create --print-join-command
```

Now get the kubeadm join command and add --ignore-preflight-errors=all

It should look like something like this... 
```
kubeadm join kind-control-plane:6443 --token <bootstrap-token here>     --discovery-token-ca-cert-hash <<sha256:ca-cert-hash>>
```

bash into worker nodes and repeat the process
```
docker ps
docker exec -it kind-worker bash
kubeadm join kind-control-plane:6443 --token << >>     --discovery-token-ca-cert-hash sha256:<< >> --ignore-preflight-errors=all
```

If you have two sessions, bash into control plane and check the nodes

```
kubectl get nodes
```
If all went well, you should be seeing additional nodes in the cluster. 


### Configure local kubectl 
come out of the container shell. 

Try
```
kubectl get nodes
```

This above step wont work.
you need to export the config from the inside the kind cluster and update the kubectl locally to access the cluster. 

```
kind export kubeconfig
kubectl get nodes

```
Now, you should see the all the nodes. 


### Understand static pods

Run the following commands and understand what is going on here..
```
kubectl get pods -n kube-system
kubectl get pods -n kube-system -o wide
```
First, Under the hood, the kubeadm runs the static pods to initalize the core components. 

The static pod manifests are stored in 
```
cd /etc/kubernetes/manifests
root@kind-control-plane:/etc/kubernetes/manifests# ls -ltr
total 16
-rw------- 1 root root 3692 Nov  8 02:05 kube-apiserver.yaml
-rw------- 1 root root 3380 Nov  8 02:05 kube-controller-manager.yaml
-rw------- 1 root root 1384 Nov  8 02:05 kube-scheduler.yaml
-rw------- 1 root root 2115 Nov  8 02:05 etcd.yaml
```


The static pods are managed by the kubelet not by kubectl. 

```
root@kind-control-plane:/etc/kubernetes/manifests# crictl ps
CONTAINER           IMAGE               CREATED             STATE               NAME                      ATTEMPT             POD ID
c243057a48413       1120bf0b8b414       About an hour ago   Running             calico-kube-controllers   0                   1b42e2cc45e6f
d13f699a9d5fa       bfe3a36ebd252       About an hour ago   Running             coredns                   0                   fed3ff1d74ec1
b05124159ab78       bfe3a36ebd252       About an hour ago   Running             coredns                   0                   0a721af9d88ce
df459e812730d       c1fa37765208c       About an hour ago   Running             calico-node               0                   3367c6ba943e4
5da07d698efa1       c39074f0dc90a       About an hour ago   Running             calico-typha              0                   cce832ee1b1dc
c6edb27c85729       fe7245688ff6b       About an hour ago   Running             tigera-operator           0                   24f20087a83ad
73721a2a3cb22       47e289e332426       2 hours ago         Running             kube-proxy                0                   fb89c2de09235
f2e6212be0069       0369cf4303ffd       2 hours ago         Running             etcd                      0                   cd0b0ee253952
509a0c03ce946       7dafbafe72c90       2 hours ago         Running             kube-controller-manager   0                   2a66c7631f7f8
13e8e5c471331       4d648fc900179       2 hours ago         Running             kube-scheduler            0                   6521ba3b1c11f
bb9bfbaecff60       8cba89a89aaa8       2 hours ago         Running             kube-apiserver            0                   a2ec580761074

```

How do you demonstrate that these pods are owned by the kubelet... 
```
while true ; do time ; done
```

you can open another terminal and delete the etcd , it wont affect because it is static pods. 

The other way is that you can completely rm the 

How do you identify the static pods through naming convention?

kubeadm init phase control-plane apiserver --config /kind/kubeadm.conf
Now check 
Run the following 
crictl ps
kubeadm help
Go through the kubeadm init phase --help

Kubeadm troubleshooting  or fix the things

### ETCDCTL 

```
kubectl create deployment test --image=nginx

kubectl scale deployment test --replicas=5

kubectl get pods -o wide

```

```
docker exec -it kind-control-plane bash
cd /etc/kubernetes/manifests
curl -LO git.io/etcdclient.yaml
```
Now, the etcd client pod should have come up.. 


```
crictl ps
crictl exec -it <<>> sh
which etcdctl
etcdctl
etcdctl endpoint status
```

find out where are the certs laid 
find out cert details and etc. 
```
etcdctl snapshot save --help
etcdctl snapshot save <filename>
```

you can take a look at the pod spec 

```
cat 
/var/lib/etcd
cat etcdclient.yaml 
```

```
  volumeMounts:
    - mountPath: /etc/kubernetes/pki/etcd
      name: etcd-certs
      readOnly: true
    - mountPath: /var/lib/etcd
      name: etcd-data
      readOnly: false
```
copy the back from the pod to control plane node

```
cp backup /var/lib/etcd/backup
exit
```

Now, you have snapshot is copied over to the control plane node.

```
export KUBECONFIG=/etc/kubernetes/admin.conf
kubectl get pods
```
The pods must be running.. 

Now try to break the cluster.

### Break the cluster (Yes..!)

so far, you have used etcdclient static pod to snapshot of etcd data. Now, it is in your control plane node. 

```
docker exec -it kind-control-plane bash

```


Move all the static pods to /tmp/ directory . 

```

root@kind-control-plane:/etc/kubernetes/manifests# crictl ps
CONTAINER           IMAGE               CREATED             STATE               NAME                      ATTEMPT             POD ID
d11c35d7d500f       7dafbafe72c90       54 minutes ago      Running             kube-controller-manager   18                  2a66c7631f7f8
3df122099b28e       4d648fc900179       2 hours ago         Running             kube-scheduler            17                  6521ba3b1c11f
5ed459d15abae       c1fa37765208c       2 hours ago         Running             calico-node               26                  3367c6ba943e4
c8b440487a420       c39074f0dc90a       2 hours ago         Running             calico-typha              12                  cce832ee1b1dc
781202e26b1a2       2c4adeb21b4ff       2 hours ago         Running             etcdclient                0                   d0aaf1c2d1e74
c243057a48413       1120bf0b8b414       20 hours ago        Running             calico-kube-controllers   0                   1b42e2cc45e6f
d13f699a9d5fa       bfe3a36ebd252       20 hours ago        Running             coredns                   0                   fed3ff1d74ec1
b05124159ab78       bfe3a36ebd252       20 hours ago        Running             coredns                   0                   0a721af9d88ce
c6edb27c85729       fe7245688ff6b       20 hours ago        Running             tigera-operator           0                   24f20087a83ad
73721a2a3cb22       47e289e332426       21 hours ago        Running             kube-proxy                0                   fb89c2de09235
f2e6212be0069       0369cf4303ffd       21 hours ago        Running             etcd                      0                   cd0b0ee253952
bb9bfbaecff60       8cba89a89aaa8       21 hours ago        Running             kube-apiserver            0                   a2ec580761074
root@kind-control-plane:/etc/kubernetes/manifests# 
```
you have a functioning cluster ..

Break ..

```
mv /etc/kubernetes/manifests/* /tmp/
```
Now, you should see

```
root@kind-control-plane:/etc/kubernetes/manifests# crictl ps
CONTAINER           IMAGE               CREATED             STATE               NAME                      ATTEMPT             POD ID
5ed459d15abae       c1fa37765208c       2 hours ago         Running             calico-node               26                  3367c6ba943e4
c8b440487a420       c39074f0dc90a       2 hours ago         Running             calico-typha              12                  cce832ee1b1dc
c243057a48413       1120bf0b8b414       20 hours ago        Running             calico-kube-controllers   0                   1b42e2cc45e6f
d13f699a9d5fa       bfe3a36ebd252       20 hours ago        Running             coredns                   0                   fed3ff1d74ec1
b05124159ab78       bfe3a36ebd252       20 hours ago        Running             coredns                   0                   0a721af9d88ce
c6edb27c85729       fe7245688ff6b       20 hours ago        Running             tigera-operator           0                   24f20087a83ad
73721a2a3cb22       47e289e332426       21 hours ago        Running             kube-proxy                0                   fb89c2de09235
root@kind-control-plane:/etc/kubernetes/manifests# 
```

Now, try 

```
kubectl get pods
```

you should not see any output.. 

Try to delete etcd member

```
rm -rf /var/lib/etcd/member
```

you have deleted etcd data ..

```
mv /tmp/* .
ls

```

The static pod should have registered .. 

```
root@kind-control-plane:/etc/kubernetes/manifests# crictl ps
CONTAINER           IMAGE               CREATED             STATE               NAME                      ATTEMPT             POD ID
12d2c5988267c       2c4adeb21b4ff       27 seconds ago      Running             etcdclient                0                   3c78c7a117b11
22d9dc1f2a0e6       8cba89a89aaa8       27 seconds ago      Running             kube-apiserver            0                   63aa5f1fdf14d
23cae4e98f3ae       4d648fc900179       27 seconds ago      Running             kube-scheduler            0                   90a43ed2283ac
9bcc9a2ac01dd       0369cf4303ffd       27 seconds ago      Running             etcd                      0                   e161fb8c9eba4
e9d06457c5623       7dafbafe72c90       27 seconds ago      Running             kube-controller-manager   0                   d58266b1199da
5ed459d15abae       c1fa37765208c       2 hours ago         Running             calico-node               26                  3367c6ba943e4
c8b440487a420       c39074f0dc90a       2 hours ago         Running             calico-typha              12                  cce832ee1b1dc
c243057a48413       1120bf0b8b414       20 hours ago        Running             calico-kube-controllers   0                   1b42e2cc45e6f
d13f699a9d5fa       bfe3a36ebd252       20 hours ago        Running             coredns                   0                   fed3ff1d74ec1
b05124159ab78       bfe3a36ebd252       20 hours ago        Running             coredns                   0                   0a721af9d88ce
c6edb27c85729       fe7245688ff6b       20 hours ago        Running             tigera-operator           0                   24f20087a83ad
73721a2a3cb22       47e289e332426       21 hours ago        Running             kube-proxy                0                   fb89c2de09235
root@kind-control-plane:/etc/kubernetes/manifests# 
```

It is clean now

```
kubectl get pods
```

Now break again 

```
mv /etc/kubernetes/manifests/* /tmp/
```
The pods have been deleted and clean now. 

```
crictl ps
```

Now, move etcdclient back to the static pod place

```
mv /tmp/etcdclient.yaml /etc/kubernetes/manifests/
crictl ps
crictl exec -it <<>> sh  
```

copy the backup file

```
cp /var/lib/etcd/backup .
/ # ls -ltr
total 3452
drwxr-xr-x    2 root     root         12288 Oct  1  2018 bin
drwxrwxrwt    2 root     root          4096 Oct  1  2018 tmp
drwxr-xr-x    2 nobody   nogroup       4096 Oct  1  2018 home
drwxr-xr-x    1 root     root          4096 Nov 30  2018 usr
dr-xr-xr-x   13 root     root             0 Nov  8 00:01 sys
drwxr-xr-x    1 root     root          4096 Nov  8 23:05 var
dr-xr-xr-x  233 root     root             0 Nov  8 23:05 proc
drwxr-xr-x    1 root     root          4096 Nov  8 23:05 etc
drwxr-xr-x    5 root     root           360 Nov  8 23:05 dev
drwx------    1 root     root          4096 Nov  8 23:07 root
-rw-r--r--    1 root     root       3493920 Nov  8 23:08 backup
```
Delete the etcd contents now
```
rm -rf /var/lib/etcd/member/
```
restore the backup 
```
/ # ls -ltr
total 3456
drwxr-xr-x    2 root     root         12288 Oct  1  2018 bin
drwxrwxrwt    2 root     root          4096 Oct  1  2018 tmp
drwxr-xr-x    2 nobody   nogroup       4096 Oct  1  2018 home
drwxr-xr-x    1 root     root          4096 Nov 30  2018 usr
dr-xr-xr-x   13 root     root             0 Nov  8 00:01 sys
drwxr-xr-x    1 root     root          4096 Nov  8 23:05 var
dr-xr-xr-x  233 root     root             0 Nov  8 23:05 proc
drwxr-xr-x    1 root     root          4096 Nov  8 23:05 etc
drwxr-xr-x    5 root     root           360 Nov  8 23:05 dev
drwx------    1 root     root          4096 Nov  8 23:07 root
-rw-r--r--    1 root     root       3493920 Nov  8 23:08 backup
drwx------    3 root     root          4096 Nov  8 23:11 default.etcd
/ # 
```
move the etcd

```
mv default.etcd/member/ /var/lib/etcd/
ls -al /var/lib/etcd/member/
total 16
drwx------    4 root     root          4096 Nov  8 23:11 .
drwx------    3 root     root          4096 Nov  8 23:12 ..
drwx------    2 root     root          4096 Nov  8 23:11 snap
drwx------    2 root     root          4096 Nov  8 23:11 wal
```
move the pods to manifests folder 

```
mv /tmp/* /etc/kubernetes/manifests/
```

Now, check the static pods. 

```
kubectl get pods
```

Try to scale the deployment to replicas to 2.



```
kubectl scale deploy test --replicas 2
```


### Upgrade kubeadm 

Links are 
```
cd 
curl -LO https://dl.k8s.io/v1.19.2/bin/linux/amd64/kubeadm
curl -LO https://dl.k8s.io/v1.19.2/bin/linux/amd64/kubelet

cd 
chmod +x k*
```

now, explore kubeadm explore plan

```
kubeadm --help
kubeadm upgrade --help
kubeadm upgrade plan
```

The upgrade plan command will give you 

```
COMPONENT                 CURRENT    AVAILABLE
kube-apiserver            v1.19.1    v1.19.3
kube-controller-manager   v1.19.1    v1.19.3
kube-scheduler            v1.19.1    v1.19.3
kube-proxy                v1.19.1    v1.19.3
CoreDNS                   1.7.0      1.7.0
etcd                      3.4.13-0   3.4.13-0
```

Now, Apply kubeadm upgrade apply v1.19.3


```
kubeadm upgrade apply v1.19.3
```

The output would like this 

```
root@kind-control-plane:/# kubeadm upgrade apply v1.19.3
[upgrade/config] Making sure the configuration is correct:
[upgrade/config] Reading configuration from the cluster...
[upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[preflight] Running pre-flight checks.
[upgrade] Running cluster health checks
[upgrade/version] You have chosen to change the cluster version to "v1.19.3"
[upgrade/versions] Cluster version: v1.19.1
[upgrade/versions] kubeadm version: v1.19.1
[upgrade/version] FATAL: the --version argument is invalid due to these errors:

        - Specified version to upgrade to "v1.19.3" is higher than the kubeadm version "v1.19.1". Upgrade kubeadm first using the tool you used to install kubeadm

Can be bypassed if you pass the --force flag
To see the stack trace of this error execute with --v=5 or higher
root@kind-control-plane:/# 

```

Take a moment and read through the output ..

```
kubeadm upgrade apply v1.19.3 --force
```

The output 

```
[upgrade/config] Making sure the configuration is correct:
[upgrade/config] Reading configuration from the cluster...
[upgrade/config] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[preflight] Running pre-flight checks.
[upgrade] Running cluster health checks
[upgrade/version] You have chosen to change the cluster version to "v1.19.3"
[upgrade/versions] Cluster version: v1.19.1
[upgrade/versions] kubeadm version: v1.19.1
[upgrade/version] Found 1 potential version compatibility errors but skipping since the --force flag is set: 

        - Specified version to upgrade to "v1.19.3" is higher than the kubeadm version "v1.19.1". Upgrade kubeadm first using the tool you used to install kubeadm
[upgrade/prepull] Pulling images required for setting up a Kubernetes cluster
[upgrade/prepull] This might take a minute or two, depending on the speed of your internet connection
[upgrade/prepull] You can also perform this action in beforehand using 'kubeadm config images pull'
[upgrade/apply] Upgrading your Static Pod-hosted control plane to version "v1.19.3"...
Static pod: kube-apiserver-kind-control-plane hash: 31acd4150f7230e8323fdfb35067e3aa
Static pod: kube-controller-manager-kind-control-plane hash: 411ac4af029477effdb68d633678044e
Static pod: kube-scheduler-kind-control-plane hash: d8964234650b330c55fcf8fb2f5295dd
[upgrade/etcd] Upgrading to TLS for etcd
[upgrade/etcd] Non fatal issue encountered during upgrade: the desired etcd version "3.4.13-0" is not newer than the currently installed "3.4.13-0". Skipping etcd upgrade
[upgrade/staticpods] Writing new Static Pod manifests to "/etc/kubernetes/tmp/kubeadm-upgraded-manifests082109356"
[upgrade/staticpods] Preparing for "kube-apiserver" upgrade
[upgrade/staticpods] Renewing apiserver certificate
[upgrade/staticpods] Renewing apiserver-kubelet-client certificate
[upgrade/staticpods] Renewing front-proxy-client certificate
[upgrade/staticpods] Renewing apiserver-etcd-client certificate
[upgrade/staticpods] Moved new manifest to "/etc/kubernetes/manifests/kube-apiserver.yaml" and backed up old manifest to "/etc/kubernetes/tmp/kubeadm-backup-manifests-2020-11-08-23-59-50/kube-apiserver.yaml"
[upgrade/staticpods] Waiting for the kubelet to restart the component
[upgrade/staticpods] This might take a minute or longer depending on the component/version gap (timeout 5m0s)
Static pod: kube-apiserver-kind-control-plane hash: 31acd4150f7230e8323fdfb35067e3aa
```

Try this 
```
kubectl get nodes --kubeconfig /etc/kubernetes/admin.conf 
```

you may see different versions reported

```
root@kind-control-plane:/# kubectl get nodes --kubeconfig /etc/kubernetes/admin.conf 
NAME                 STATUS   ROLES    AGE   VERSION
kind-control-plane   Ready    master   21h   v1.19.1
kind-worker          Ready    <none>   20h   v1.19.1
kind-worker2         Ready    <none>   20h   v1.19.1
kind-worker3         Ready    <none>   20h   v1.19.1
```

```
kubectl version --kubeconfig /etc/kubernetes/admin.conf
```
```
root@kind-control-plane:/# kubelet --version
Kubernetes v1.19.1
root@kind-control-plane:/# ./kubelet --version
Kubernetes v1.19.2
root@kind-control-plane:/# 
```
move the kubelet version and replace the new one

```
mv kubelet /usr/bin/kubelet
systemctl restart kubelet
```

```
root@kind-control-plane:/# kubectl get nodes --kubeconfig /etc/kubernetes/admin.conf

NAME                 STATUS   ROLES    AGE   VERSION
kind-control-plane   Ready    master   22h   v1.19.2
kind-worker          Ready    <none>   21h   v1.19.1
kind-worker2         Ready    <none>   20h   v1.19.1
kind-worker3         Ready    <none>   20h   v1.19.1
```

Now, the version reported by the kubelet is ... 


Now, you copy the kubelet to the local and repeat the steps for other worker nodes

```
docker cp kind-control-plane:/usr/bin/kubelet .
```

To copy to the worker node

```
docker cp kind-worker:/usr/bin/kubelet .
docker cp kind-worker2:/usr/bin/kubelet .
docker cp kind-worker3:/usr/bin/kubelet .
```

```
kubectl get nodes
```
Now restart the kubelet to see the latest version

```
docker exec -it kind-worker systemctl restart kubelet
docker exec -it kind-worker2 systemctl restart kubelet
docker exec -it kind-worker3 systemctl restart kubelet
```

If the above step does not work, bash into the container and download the kubectl 
replace and restart the service..

### explain

```
kubectl explain pod.spec
kubectl explain pod.spec --recursive
kubectl explain pod.spec.initContainers
kubectl api-resources
```