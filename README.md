# kind-scenarios

The objective of this repo is to create a local kind cluster with kubeadm script. Thanks to mauilion

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
controlPlaneEndpoint: ** kind-control-plane:6443
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
