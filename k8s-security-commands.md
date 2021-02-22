
```
kubectl config view --kubeconfig my-kube-config
```

I would like to use the dev-user to access test-cluster-1. Set the current context to the right one so I can do that.

kubectl config --kubeconfig=/root/my-kube-config use-context research 

```
kubectl config --help
```


### Role / Role Binding

To findout the group information for the role, you need to look at the rolebinding 

```
kubectl -n kube-system describe rolebining <<rolename>>

kubectl -n kube-system describe rolebining kube-proxy

kubectl get pods --as dev-user
```

kubectl create role developer --verb=list --verb=create --resource=pods --dry-run=client -o yaml > role-developer.yaml

kubectl create role developer --verb=list --verb=create --resource=pods --dry-run=client -o yaml > role-developer.yaml

kubectl create rolebinding dev-user-binding --role developer --user dev-user --dry-run=client -o yaml > role-binding.yaml

kubectl -n blue get rolebinding dev-user-binding -o yaml > blue-role-binding.yaml

Where do you see the api-groups

kubectl auth can-i list nodes --as michelle

What do you need to ni

```
kubectl create clusterrole node-admin --verb=get,watch,list,create,delete  --resource=nodes --dry-run=client -o yaml > node-admin-role.yaml

kubectl create clusterrolebinding michelle-binding --clusterrole node-admin --user michelle --dry-run=client -o yaml > role-binding.yaml

kubectl auth can-i list nodes --as michelle

kubectl create clusterrole storage-admin  --verb=get,watch,list,create,delete  --resource=persistentvolumes,storageclasses --dry-run=client -o yaml > storage-admin-role.yaml

kubectl create clusterrolebinding michelle-storage-admin --clusterrole storage-admin --user michelle --dry-run=client -o yaml > m.yaml

kubectl auth can-i list persistentvolumes --as michelle

```

```
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: node-admin
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["get", "watch", "list", "create", "delete"]

---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: michelle-binding
subjects:
- kind: User
  name: michelle
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: node-admin
```

In the above command why is user is in the apigroup `rbac.authorization.k8s.io`

For storage 

---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: storage-admin
rules:
- apiGroups: [""]
  resources: ["persistentvolumes"]
  verbs: ["get", "watch", "list", "create", "delete"]
- apiGroups: ["storage.k8s.io"]
  resources: ["storageclasses"]
  verbs: ["get", "watch", "list", "create", "delete"]

---


```
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: michelle-storage-admin
subjects:
- kind: User
  name: michelle
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: storage-admin
  apiGroup: rbac.authorization.k8s.io
```

#### Secret

```
kubectl create secret docker-registry private-reg-cred --docker-username=dock_user --docker-password=dock_password --docker-server=myprivateregistry.com:5000 --docker-email=dock_user@myprivateregistry.com

from (kubernetes.io)
kubectl create secret docker-registry secret-tiger-docker \
  --docker-username=tiger \
  --docker-password=pass113 \
  --docker-email=tiger@acme.com
```

### Security Context

```
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper
  namespace: default
spec:
  securityContext:
    runAsUser: 1010
  containers:
  - command:
    - sleep
    - "4800"
    image: ubuntu
    name: ubuntu-sleeper
```

Set date 
```
 kubectl exec -it ubuntu-sleeper -- date -s '19 APR 2012 11:14:00'
```

Run root user with SYS_TIME capability
```
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper
  namespace: default
spec:
  containers:
  - command:
    - sleep
    - "4800"
    image: ubuntu
    name: ubuntu-sleeper
    securityContext:
      capabilities:
        add: ["SYS_TIME"]
```
??? Can i edit pod spec ... 

### Network policies

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: internal-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      name: internal
  policyTypes:
  - Egress
  - Ingress
  ingress:
    - {}
  egress:
  - to:
    - podSelector:
        matchLabels:
          name: mysql
    ports:
    - protocol: TCP
      port: 3306

  - to:
    - podSelector:
        matchLabels:
          name: payroll
    ports:
    - protocol: TCP
      port: 8080

  - ports:
    - port: 53
      protocol: UDP
    - port: 53
      protocol: TCP
```