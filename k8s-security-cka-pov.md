# Security 

## Objective: <br>

The intent is to capture the sub-topics and contents and branch-off to different topics from here. 

### Protect kube-api servers

1. Who can access (Admin, Developers, Appln End users, Service Accounts {bots})
2. What can you do with this?. (Entitlement)

### Authentication - Access Modes

Users / Service Accounts - {Think about different users who can access the system}

1. Username/password (static files through csv)  
     ***kube-api server configuration*** 
     ``` 
       kube-apiserver.service
     --basic-auth-file=user-details.csv
     
     /etc/kubenernetes/manifests/kube-apiserver.yaml

    spec:
      containers:
        - command:
         -kube-apiserver
         - --authorization-mode=Node, RBAC
         ...
         - --basic-auth-file=user-details.csv
     ``` 
     How do you pass the user information 
     ```
     curl -v -k https://kube-api-server:6443/api/v1/pods -u "user1:password123"
     ```
     ```
     user csv
     --------
     password,user1,u001,group1
     password,user2,u002,group2
     ```
2. Username/tokens <br>
    ***kube-api server configuration*** 
    For tokens (Same as above)
     ```
     curl -v -k https://kube-api-server:6443/api/v1/pods --header "Authorization: Bearer w45fgdfgdfgdfg...."
     ```
     ```
     static token csv
     --------
     w45fgdfgdfgdfg....,user1,u001,group1
     w4tyutyu5fsdrr....,user2,u002,group2
     ```


3. Certificates
4. External Auth Providers: LDAP <br>
5. Service Accounts (services)


K8s does not manage user accounts internally and you cannot create users locally. 
Only service accounts can be created. 

Requests are send through kube-apiservers. 

### Authorization 

1. RBAC 
2. ABAC
3. Node Auth 
4. Webhook mode (OPA)

### TLS Certificates

Link to the diagram and Certs

**To Kube-api server**

1. Admin -> kube-api
2. Kube Controller Manager -> Kube-api
3. Kube proxy -> Kube-api
4. kube scheduler -> Kube-api



**From Kube-api server** 
1. To Etcd server
2. To kubelet server

Refer kubernetes client certificates in the notes. 