
To create a pod & dry run
```
kubectl run nginx-pod --image=nginx:alpine --dry-run=client -o yaml > n1-test-1.yaml
```

Create a pod with labels 
```
kubectl run messaging --image=redis:alpine -l tier=msg --dry-run=client -o yaml > r1-test-1.yaml
kubectl get pods --show-labels
kubectl create ns apx-x9984674
kubectl get nodes -o json
```



To expose a pod 
```
kubectl expose pod messaging --name messaging-service --port 6379 --target-port 6379 --dry-run=client -o yaml > ser-test-1.yaml
```


To create deployment
```
kubectl create deployment hr-web-app --image=kodekloud/webapp-color --replicas=2 --dry-run=client -o yaml > depl-hr-test-1.yaml
```

Imperative deployment scaling
```
kubectl scale deployment hr-web-app --replicas=2
```


To create pod with command 
```
 kubectl run static-busybox --image=busybox -o yaml --dry-run=client --command sleep 1000 > static-pod.yaml
```


**<u>Kubelet</u>**
Review 
```
cd /var/lib/kubelet

root@kind-control-plane:/var/lib/kubelet# ls -ltr
total 36
-rw-r--r--  1 root root  214 Nov 22 19:28 kubeadm-flags.env
-rw-r--r--  1 root root  912 Nov 22 19:28 config.yaml
drwxr-xr-x  2 root root 4096 Nov 22 19:28 pki
drwxr-x---  2 root root 4096 Nov 22 19:28 plugins_registry
drwxr-x---  2 root root 4096 Nov 22 19:28 plugins
-rw-------  1 root root   62 Nov 22 19:28 cpu_manager_state
drwxr-x---  2 root root 4096 Nov 22 19:29 pod-resources
drwxr-xr-x  2 root root 4096 Nov 27 19:56 device-plugins
drwxr-x--- 14 root root 4096 Nov 27 19:56 pods


Look at the config.yaml 
`staticPodPath: /etc/kubernetes/manifests`


````

