## K8 Security Basics 
<br>

### Introduction

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

### Role based access control

Keywords: <mark> role, role bindings </mark>

#### Identity in the K8s.

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


#### Role / Rolebindings

<br>

Role is a abstract capabilities,
Rolebinding is assignment of role to an identities. 
(?Are there any groups - or is role is like a group?)

<br>

#### Apply Role / Rolebinding 

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


