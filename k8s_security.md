# K8 Security Basics 
<br>

## Introduction

<br>

RBAC is introduced in the K8 1.5 and GA in K8 1.8. 
Everything centered around k8s API server. 
RBAC provides the mechanism for <mark> restricting both access and actions on K8s API server.  </mark> 

RBAC is the keypiece on how to provide the right level of access to the folks. 

Every request is authenticated to the API server.
Authentication provides the identity of the caller. (Service)

K8s does not have identity store and Pluggable Authentication provider can be attached. (Active directory) Through webhooks

Authorization is a combination of the identity of the user, the resource (effectively the HTTP path), and the verb or action the user is attempting to perform.

User --> Resource (which resource?) --> Actions? (what kind of actions?)

## Role based access control

Keywords: <mark> role, role bindings </mark>

### Identity in the K8s.

Keywords: <mark> service account identity, user identity </mark>

+ Request comes with identity
+ system:unauthenticated 
+ <mark>Service accounts for running K8s components </mark> and are managed within K8s itself. 
+ <mark>User accounts are users of the cluster + (CD systems that runs outside of the cluster </mark>



Supported Auth providers <br>
+ Basic Auth
+ x509 client certificates
+ Static token files on the bost
+ Auth providers: ex: In cloud AWS IAM
+ Authentication webhooks


### Role / Rolebindings

<br>

Role is a abstract capabilities,
Rolebinding is assignment of role to an identities. 
(?Are there any groups - or is role is like a group?)

<br>

### Apply Role / Rolebinding 

<br>
Understand what resource are namespaces and what are not namespaced. 
<br>
<br>

keywords: <mark>Role, Rolebinding, ClusterRole, ClusterRolebinding </mark>

```
# In a namespace
kubectl api-resources --namespaced=true

# Not in a namespace
kubectl api-resources --namespaced=false

```

Role and Rolebindings are namespaced
ClusterRole, ClusterRoleBindings are not namespaced. 

The question that you need to ask ... when do you use Role, RoleBindings and ClusterRole, ClusterRoleBindings ?


### Verbs for K8s Roles

Create, delete, get, list, patch, update, watch, proxy

### Using Built-in Roles

```
kubectl get clusterroles
```
The cluster-admin role provides complete access to the entire cluster.

The admin role provides complete access to a complete namespace.

The edit role allows an end user to modify things in a namespace.

The view role allows for read-only access to a namespace.

### AUTO-RECONCILIATION OF BUILT-IN ROLES

The roles gets reset whenever K8s API server boots up, so to prevent from happenging, rbac.authorization.kubernetes.io/autoupdate annotation with a value of false to the built-in ClusterRole resource. 

<br>

By default, the Kubernetes API server installs a cluster role that allows system:unauthenticated users access to the API server’s API discovery endpoint. 

```
--anonymous-auth=false flag is set on your API server.
```

### Techniques for Managing RBAC

### Testing Authorization with can-i

```
kubectl auth can-i create pods
kubectl auth can-i get pods --subresource=logs
```

### Managing RBAC in Source Control

```
kubectl auth reconcile -f some-rbac-config.yaml

To print
kubectl auth reconcile -f some-rbac-config.yaml --dry-run
```

## Advanced Topics

### Aggegating roles

aggregation rule to combine multiple roles together in a new role..


A best practice for managing ClusterRole resources is to create a number of fine-grained cluster roles and then aggregate them together to form higher-level or broadly defined cluster roles. 

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: edit
  ...
aggregationRule:
  clusterRoleSelectors:
  - matchLabels:
      rbac.authorization.k8s.io/aggregate-to-edit: "true"
...

```
ClusterRole objects that have a label of rbac.authorization.k8s.io/aggregate-to-edit set to true.

### Using Groups for Bindings
best practice to use groups to manage the roles

bind a group to a ClusterRole or a namespace Role, anyone who is a member of that group gains access to the resources and verbs defined by that role. 

To bind a group to a ClusterRole you use a Group kind for the subject in the binding:

```
...
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: Group
  name: my-great-groups-name
...
```


## Private Registry (Image Security)

```
kubectl create secret docker-registry regcred \
    --docker-server= <private-registry.io>  \
    --docker-username=registry-user \
    --docker-password=registry-password \
    --docker-email=registry@private.com
```

Here in the pod spec pass imagePullSecrets

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
    - name: 
      image:
   imagePullSecrets:
    - name: regcred
```

Example: 

```
kubectl create secret docker-registry private-reg-cred --docker-username=dock_user --docker-password=dock_password --docker-server=myprivateregistry.com:5000 --docker-email=dock_user@myprivateregistry.com
```

## Security Contexts

'Id' of the user to run to the container

Linux capability

```
docker run --user=1001 ubuntu sleep 1000
```

Security at container level or pod level

look at the securityContext 

securityContext @ Pod level

```
apiVerion: v1
kind: Pod
meta-data:
  name: web-pod
spec:
  securityContext:
    runAsUser: 1000
  containers:
    - name: ubuntu
      image: ubuntu
      command: ["sleep","3600"]  
```

securityContext @ container level

```
apiVerion: v1
kind: Pod
meta-data:
  name: web-pod
spec:
  containers:
    - name: ubuntu
      image: ubuntu
      command: ["sleep","3600"]
```

Try

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


### Network policy

How do you restrict the network inbounds from certain pods?

```
apiVersion:
kind:
metadata:
spec:
  podSelector:
    matchLables:
      role: db
  policyTypes:
    - Ingress
  ingress:
    - from:
      - podSelector:
          matchLabels:
            name: api-pod
      ports:
        - protocol: TCP
          port: 3340
```