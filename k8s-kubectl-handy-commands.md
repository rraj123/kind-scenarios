# K8s - Kubectl Commands 

### Introduction
Collect and come with a list of imperative commands.. 

```
kubectl config use-context

(with the labels )
kubectl run messaging --image=redis:alpine -l tier=msg --dry-run=client -o yaml

kubectl get pods --show-labels

(exposing pod)
kubectl expose pod messaging --name messaging-service --port 6379 --target-port 6379  --dry-run=client -o yaml 

kubectl create deployment hr-web-app --image=kodekloud/webapp-color --replicas=

```



#### Test 3 ####

```
kubectl create sa pvviewer

kubectl create clusterrole pvviewer-role --verb=list --resource=persistentvolumes --dry-run=client -o yaml 

kubectl create clusterrolebinding pvviewer-role-binding --clusterrole=pvviewer-role --serviceaccount=default:pvviewer --dry-run=client -o yaml 

kubectl run pvviewer --image=redis --serviceaccount=pvviewer --dry-run=client -o yaml 
```

Internal IP of all Nodes of the cluster and save the result
```
kubectl get nodes -o jsonpath='{.items[*].status.addresses[?(@.type=="InternalIP")].address} '

kubectl get nodes -o jsonpath='{.items[*].status.addresses[].address} '

kubectl run alpha --image=nginx --command sleep 4800 --env="name=alpha"  --dry-run=client -o yaml > multi-pod.yaml

```

```
kubectl run lion --image=redis:alpine --limits='cpu=200m,memory=512Mi'  --dry-run=client -o yaml

kubectl run test-conn --image=busybox --rm -it -- sh
nc -z -c
```


you can override the kubeconfig and provide other kube config file to kubectl 

```
kubectl cluster-info --kubeconfig=/root/super.kubeconfig
```

kubectl exec --stdin --tty shell-demo -- /bin/bash
kubectl exec --stdin --tty shell-demo -- /bin/bash -c env