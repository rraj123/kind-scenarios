## RBAC

How do you create role? 
Create a role object

### Role / RoleBindings (Namespaced)
```

apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata: 
  namespace: 
  name: pods-and-services
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["list","get","create","update","delete"]
  - apiGroups: [""]
    resources: ["ConfigMap"]
    verbs: ["list","get","create","update","delete"]

```
 For Core group -> apigroup must be empty.. 

RoleBinding object to link Role to 

```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  namespace: default
  name: pods-and-services
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: alice
- apiGroup: rbac.authorization.k8s.io
  kind: Group
  name: mydevs
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: pod-and-services
  
```

The key attributes are subjects and roleref

To view

```
kubectl get roles
kubectl get rolebinding
kubectl describe role pods-and-services
kubectl describe rolebinding <>


```

To check Access

```
kubectl auth can-i create deployments

To impersonate as user
kubectl auth can-i create pods --as dev-user --namespace test
```

In addition, you can restrict the role to specific set of resources...
<b>resourceNames</b>


```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata: 
  namespace: 
  name: pods-and-services
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["list","get","create","update","delete"]
  - apiGroups: [""]
    resources: ["ConfigMap"]
    verbs: ["list","get","create","update","delete"]
    resourceNames: ["blue","orange"]
```

### ClusterRole / ClusterRoleBindings 
Nodes are not namespaced..
They are cluster wide.. 


Resources categorized as 

1. Namespaced (Pods, replicasets, jobs, roles, rolebindings...)
2. ClusterScoped (nodes, pv, clusterroles, certificatesigningRequest group and etc. )

```
kubectl api-resources --namespaced=true
kubectl api-resources --namespaced=false
```




