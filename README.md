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
