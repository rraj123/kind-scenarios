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

These commands will give you built-in roles

```
kubectl get clusterroles
kubectl get clusterrolebinding
kubectl describe clusterrole cluster-admin
kubectl describe clusterrolebinding cluster-admin
```

example:

```
apiVersion: rbac.authorization.k8s.io/v1
kind: clusterRole
meta-data: 
  name: cluster-administrator
rules:
  - apiGroups: [""]
    resources: ["nodes"]
    verbs: ["list","get","create","delete"]
```

Cluster admin role

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata: 
  name: cluster-admin-role-binding
subjects:
  - kind: User
    name: cluster
```

```
kubectl get clusterrole cluster-admin -o yaml | pbcopy

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  creationTimestamp: "2020-11-12T02:39:49Z"
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
    manager: kube-apiserver
    operation: Update
    time: "2020-11-12T02:39:49Z"
  name: cluster-admin
  resourceVersion: "47"
  selfLink: /apis/rbac.authorization.k8s.io/v1/clusterroles/cluster-admin
rules:
- apiGroups:
  - '*'
  resources:
  - '*'
  verbs:
  - '*'
- nonResourceURLs:
  - '*'
  verbs:
  - '*'

```

<br>
Cluster role binding 
<br>

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  creationTimestamp: "2020-11-12T02:39:50Z"
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
  managedFields:
  - apiVersion: rbac.authorization.k8s.io/v1
    manager: kube-apiserver
    operation: Update
    time: "2020-11-12T02:39:50Z"
  name: cluster-admin
  resourceVersion: "104"
  selfLink: /apis/rbac.authorization.k8s.io/v1/clusterrolebindings/cluster-admin
  uid: 901c8a36-bd79-4840-9435-19baeedade6f
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: Group
  name: system:masters
```

** you can also create a clusterrole for the namespaced role also

So, when you create a clusterrole for namespaced resource, you get access to all  namespaces.

By default, K8s creates a bunch of built-in clusterroles. you can examine using the above commands. 

