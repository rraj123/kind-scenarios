
#### Pre-Setup
```
alias k=kubectl
k version --short # tested with v1.19.0
```

```
export do="--dry-run=client -o yaml"
k run pod1 --image=nginx $do
```
-------------
##### ConfigView #####  

Understand `kubectl config` . 
To interface with kubeconfig files. 

Understand config view 

```
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://127.0.0.1:50703
  name: kind-kind
contexts:
- context:
    cluster: docker-desktop
    user: docker-desktop
  name: docker-desktop
- context:
    cluster: kind-kind
    user: kind-kind
  name: kind-kind
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kind-kind
kind: Config
preferences: {}
users:
- name: kind-kind
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED

```



```
kubectl config view 

kubectl config get-contexts
kubectl config view -o jsonpath="{.contexts}" -o json
config view -o jsonpath="{.contexts[*].name}" | tr " " "\n" # new lines
k config view -o jsonpath="{.contexts[*].name}" | tr " " "\n" > /opt/course/1/contexts 
kubectl config current-context

cat ~/.kube/config | grep current


cat ~/.kube/config | grep current | sed -e "s/current-context: //"
---
switch config 
kubectl config use-context k8s-c1-H

```


#### Place a pod on master node #### 
Create a single Pod of image httpd:2.4.41-alpine in Namespace default. The Pod should be named pod1 and the container should be named pod1-container. This Pod should only be scheduled on a master node, do not add new labels any nodes.


```
kubectl describe node cluster1-master1 | grep Taint
k describe node cluster1-master1 | grep Labels -A 10

```
Create a pod
```
kubectl run pod1 --image=httpd:2.4.41-alpine --dry-run -o yaml > 2.yaml
```

https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/

```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: pod1
  name: pod1
spec:
  containers:
  - image: httpd:2.4.41-alpine
    name: pod1-container                  # change
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  tolerations:                            
  - effect: NoSchedule                    
    key: node-role.kubernetes.io/master   
  nodeSelector:                           
    node-role.kubernetes.io/master: ""    
status: {}
```

#### Stateful set

Scale down the stateful set.. and `record it`

```
k -n project-c13 scale sts o3db --replicas 1 --record
```

The --record created an annotation:

```

```
#### Pod is ready if service 

readiness probe
```
k8s@terminal:~$ cat ready-ser.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: ready-if-service-ready
  name: ready-if-service-ready
spec:
  containers:
  - image: nginx:1.16.1-alpine
    name: ready-if-service-ready
    startupProbe:
      exec:
        command:
          - sh
          - c
          - 'sleep 3 && true'
    readinessProbe:
      exec:
        command:
          - sh
          - c
          - 'wget -T2 -o- http://service-am-i-ready:80'
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
k8s@terminal:~$ 
```

```
# 4_pod1.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: ready-if-service-ready
  name: ready-if-service-ready
spec:
  containers:
  - image: nginx:1.16.1-alpine
    name: ready-if-service-ready
    resources: {}
    startupProbe:                               # add from here
      exec:
        command:
        - sh
        - -c
        - 'sleep 3 && true'
    readinessProbe:
      exec:
        command:
        - sh
        - -c
        - 'wget -T2 -O- http://service-am-i-ready:80'   # to here
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

#### kubectl sorting
```
kubectl get pod -A --sort-by=.status.startTime
```
#### PV, PVC, Deployment. 

Create a new PersistentVolume named safari-pv. It should have a capacity of 2Gi, accessMode ReadWriteOnce, hostPath /Volumes/Data and no storageClassName defined.

Next create a new PersistentVolumeClaim in Namespace project-tiger named safari-pvc . It should request 2Gi storage, accessMode ReadWriteOnce and should not define a storageClassName. The PVC should bound to the PV correctly.

Finally create a new Deployment safari in Namespace project-tiger which mounts that volume at /tmp/safari-data. The Pods of that Deployment should be of image httpd:2.4.41-alpine.

create a pv


```
kubectl create deployment safari --image=httpd:2.4.41-alpine --namespace=project-tiger --dry-run=client -o yaml > depl.yaml
```

PV
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: safari-pv 
spec:
  capacity:
    storage: 2Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /Volumes/Data
```
PVC
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
   name: safari-pvc
   namespace: project-tiger
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
```

```
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: safari
  name: safari
  namespace: project-tiger
spec:
  replicas: 1
  selector:
    matchLabels:
      app: safari
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: safari
    spec:
      containers:
      - image: httpd:2.4.41-alpine
        name: httpd
        resources: {}
        volumeMounts:
        - name: data
          mountPath: /tmp/safari-data
      volumes:
      - name: data
        persistentVolumeClaim:
          claimName: safari-pvc
```

#### Node and Pod Resource Usage ####

The metrics-server hasn't been installed yet in the cluster, but it's something that should be done soon. Your college would already like to know the kubectl commands to:

show node resource usage
show Pod and their containers resource usage

```
kubectl top -h
kubectl top node
kubectl top pod -h
```

#### Get Master Information #### 

kubelet is controlled via systemd
```
ps aux | grep kubelet
find /etc/systemd/system/ | grep kube
```
not any other components
```
find /etc/systemd/system/ | grep etcd
```

All other core components are maintained in 

```
find /etc/kubernetes/manifests/
```

Actually, let's check all Pods running on in the kube-system Namespace on the master node:

```
kubectl -n kube-system get pod -o wide | grep master1
kubectl -n kube-system get ds
kubectl -n kube-system get deploy
```

#### Kill Scheduler, Manual Scheduling #### 

#### RBAC ServiceAccount Role RoleBinding

Create a new ServiceAccount processor in Namespace project-hamster. Create a Role and RoleBinding, both named processor as well. These should allow the new SA to only create Secrets and ConfigMaps in that Namespace.

```
Because of this there a 4 different RBAC combinations and 3 valid ones:

Role + RoleBinding (available in single Namespace, applied in single Namespace)

ClusterRole + ClusterRoleBinding (available cluster-wide, applied cluster-wide)

ClusterRole + RoleBinding (available cluster-wide, applied in single Namespace)

Role + ClusterRoleBinding (NOT POSSIBLE: available in single Namespace, applied cluster-wide)

 
```
apiVersion: v1
kind: Namespace
metadata:
  name: <insert-namespace-name-here>

```
kubectl create ns project-hamster 
kubectl create sa processor -n project-hamster
--- or --
kubectl apply -f - <<EOF
apiVersion: v1
kind: ServiceAccount
metadata:
  name: processor
  namespace: project-hamster
EOF
--

kubectl create role processor -n project-hamster --verb=create --resource=secret --resource=configmap --dry-run=client -o yaml > role.yaml

```

Watch this belew
```
k8s@terminal:~$ kubectl create rolebinding processor -n project-hamster --role processor --serviceaccount processor  --dry-run=client -o yaml 
error: serviceaccount must be <namespace>:<name>
k8s@terminal:~$ kubectl create rolebinding processor -n project-hamster --role processor --serviceaccount project-hamster:processor  --dry-run=client -o yaml 
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  creationTimestamp: null
  name: processor
  namespace: project-hamster
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: processor
subjects:
- kind: ServiceAccount
  name: processor
  namespace: project-hamster
```

To test 

Start with this command `kubectl auth can-i -h`


```
kubectl -n project-hamster auth can-i create pod --as system:serviceaccount:project-hamster:processor

kubectl -n project-hamster auth can-i create secret \
  --as system:serviceaccount:project-hamster:processor

 kubectl -n project-hamster auth can-i create pod \
  --as system:serviceaccount:project-hamster:processor


```

### Daemonset

There is no imperative command to generate daemonset object, 
use `deploy` to generate and replace with `Daemonset`

https://v1-16.docs.kubernetes.io/docs/concepts/configuration/assign-pod-node/#affinity-and-anti-affinity

```
k -n project-tiger create deployment \
  --image=nginx:1.17.6-alpine deploy-important $do > 12.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    id: very-important                  # change
  name: deploy-important
  namespace: project-tiger              # important
spec:
  replicas: 3                           # change
  selector:
    matchLabels:
      id: very-important                # change
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        id: very-important              # change
    spec:
      containers:
      - image: nginx:1.17.6-alpine
        name: container1                # change
        resources: {}
      - image: kubernetes/pause         # add
        name: container2                # add
      affinity:                                             # add
        podAntiAffinity:                                    # add
          requiredDuringSchedulingIgnoredDuringExecution:   # add
          - labelSelector:                                  # add
              matchExpressions:                             # add
              - key: id                                     # add
                operator: In                                # add
                values:                                     # add
                - very-important                            # add
            topologyKey: kubernetes.io/hostname             # add
status: {}
```

#### Multi Containers and Pod shared Volume ####  

***This is a good one to play with***

```
kubectl run multi-container-playground --image=nginx:1.17.6-alpine --dry-run=client -o yaml > 13.yaml
k exec multi-container-playground -c c1 -- env | grep MY
k logs multi-container-playground -c c3
```

```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: multi-container-playground
  name: multi-container-playground
spec:
  containers:
  - image: nginx:1.17.6-alpine
    name: c1                                                                      # change
    resources: {}
    env:                                                                          # add
    - name: MY_NODE_NAME                                                          # add
      valueFrom:                                                                  # add
        fieldRef:                                                                 # add
          fieldPath: spec.nodeName                                                # add
    volumeMounts:                                                                 # add
    - name: vol                                                                   # add
      mountPath: /vol                                                             # add
  - image: busybox:1.31.1                                                         # add
    name: c2                                                                      # add
    command: ["sh", "-c", "while true; do date >> /vol/date.log; sleep 1; done"]  # add
    volumeMounts:                                                                 # add
    - name: vol                                                                   # add
      mountPath: /vol                                                             # add
  - image: busybox:1.31.1                                                         # add
    name: c3                                                                      # add
    command: ["sh", "-c", "tail -f /vol/date.log"]                                # add
    volumeMounts:                                                                 # add
    - name: vol                                                                   # add
      mountPath: /vol                                                             # add
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  volumes:                                                                        # add
    - name: vol                                                                   # add
      emptyDir: {}                                                                # add
status: {}
~                   
```


#### Cluster Info #### 

```
 k get node -o jsonpath="{range .items[*]}{.metadata.name} {.spec.podCIDR}{'\n'}"
 ```

 ```
 root@cluster1-master1:~# find /etc/cni/net.d/
/etc/cni/net.d/
/etc/cni/net.d/10-weave.conflist

 ```

#### Cluster Event Logging ####  

#### Namespaces and Api Resources #### 

#### Find Container of Pod and check logs #### 

```
kubectl run tiger-reunite --image=httpd:2.4.41-alpine --labels "pod=container,container=pod" -n project-tiger --dry-run=client -o yaml > tiger-re.yaml
```
------

#### Pod and Secrets ####

***Tip***
```
kubectl run secret-pod --image=busybox:1.31.1 --namespace=secret --command sh c sleep 5d --dry-run=client -o yaml

kubectl create secret generic secret2 -n secret --from-literal=user=user1 --from-literal=pass=1234
```

```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: secret-pod
  name: secret-pod
  namespace: secret                       # add
spec:
  tolerations:                            # add
  - effect: NoSchedule                    # add
    key: node-role.kubernetes.io/master   # add
  containers:
  - args:
    - sh
    - -c
    - sleep 1d
    image: busybox:1.31.1
    name: secret-pod
    resources: {}
    env:                                  # add
    - name: APP_USER                      # add
      valueFrom:                          # add
        secretKeyRef:                     # add
          name: secret2                   # add
          key: user                       # add
    - name: APP_PASS                      # add
      valueFrom:                          # add
        secretKeyRef:                     # add
          name: secret2                   # add
          key: pass                       # add
    volumeMounts:                         # add
    - name: secret1                       # add
      mountPath: /tmp/secret1             # add
      readOnly: true                      # add
  dnsPolicy: ClusterFirst
  restartPolicy: Always
  volumes:                                # add
  - name: secret1                         # add
    secret:                               # add
      secretName: secret1                 # add
status: {}
```


#### Static pod and Service ####

```
kubectl run my-static-pod \
    --image=nginx:1.16-alpine \
    --requests "cpu=10m,memory=20Mi" \
    -o yaml --dry-run=client > my-static-pod.yaml

```

***Tip***
```
kubectl expose pod my-static-pod-cluster3-master1 --name static-pod-service --type=NodePort --port 80

k get svc,ep -l run=my-static-pod

```

***Tip*** 
`look at the option -A2` very interesting command that display the two lines after the Validity

```
openssl x509  -noout -text -in /etc/kubernetes/pki/apiserver.crt | grep Validity -A2
```

```
kubeadm alpha certs check-expiration
kubeadm alpha certs check-expiration | grep apiserver
```

#### Kubelet client/server Certification ####

***Tip*** <br>
How do you know which is client and server certificates?. How do you know that these certificates are used for Client Authentication?.  This is really useful... (Investigate it .. Further..)

1. Client -> outgoing
2. Server for incoming connections.

```
openssl x509  -noout -text -in /var/lib/kubelet/pki/kubelet-client-current.pem | grep Issuer

openssl x509  -noout -text -in /var/lib/kubelet/pki/kubelet-client-current.pem | grep "Extended Key Usage" -A1

openssl x509  -noout -text -in /var/lib/kubelet/pki/kubelet.crt | grep Issuer

openssl x509  -noout -text -in /var/lib/kubelet/pki/kubelet.crt | grep "Extended Key Usage" -A1

```


#### ETCD ####

```
ETCDCTL_API=3 etcdctl snapshot save /tmp/etcd-backup.db
```

```
    - --advertise-client-urls=https://192.168.103.11:2379
    - --cert-file=/etc/kubernetes/pki/etcd/server.crt (*)
    - --client-cert-auth=true
    - --data-dir=/var/lib/etcd
    - --initial-advertise-peer-urls=https://192.168.103.11:2380
    - --initial-cluster=cluster3-master1=https://192.168.103.11:2380
    - --key-file=/etc/kubernetes/pki/etcd/server.key (*)
    - --listen-client-urls=https://127.0.0.1:2379,https://192.168.103.11:2379 (*) 
    - --listen-metrics-urls=http://127.0.0.1:2381
    - --listen-peer-urls=https://192.168.103.11:2380
    - --name=cluster3-master1
    - --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt
    - --peer-client-cert-auth=true
    - --peer-key-file=/etc/kubernetes/pki/etcd/peer.key
    - --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt (*)
    - --snapshot-count=10000
    - --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
```

```
cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep etcd
```

```
ETCDCTL_API=3  etcdctl snapshot save /tmp/etcd-backup.db \
--cacert=/etc/kubernetes/pki/etcd/ca.crt  \
--cert=/etc/kubernetes/pki/etcd/server.crt  \
--key=/etc/kubernetes/pki/etcd/server.key  
ETCDCTL_API=3 etcdctl snapshot restore /tmp/etcd-backup.db --data-dir /var/lib/etcd-backup
vim /etc/kubernetes/manifests/etcd.yaml
```


#### Range ####

```
k -n project-c13 get pod \
>   -o jsonpath="{range .items[*]} {.metadata.name}{.spec.containers[*].resources}{'\n'}"


k get pods -n project-c13 \
  -o jsonpath="{range .items[*]}{.metadata.name} {.status.qosClass}{'\n'}"


```

#### Cronjob ####

```
 k -n project-c14 create job -h
 kubectl -n project-c14 create job holy-backup-manual-run-1 --from=cronjob/holy-backup
 
```