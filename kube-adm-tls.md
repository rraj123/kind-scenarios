# Kube-adm (Kind Setup)

## Introduction:

The objective is to deconstruct the kube-adm set up and understand the internals of the kubernetes setup. 

## TLS 
Lets start with CA (Certificate authority)

```
root@kind-control-plane:/etc/kubernetes# tree
.
|-- admin.conf  (Conf file that has ca, cluster, )
|-- controller-manager.conf
|-- kubelet.conf
|-- manifests
|   |-- etcd.yaml
|   |-- kube-apiserver.yaml
|   |-- kube-controller-manager.yaml
|   `-- kube-scheduler.yaml
|-- pki
|   |-- apiserver-etcd-client.crt
|   |-- apiserver-etcd-client.key
|   |-- apiserver-kubelet-client.crt
|   |-- apiserver-kubelet-client.key
|   |-- apiserver.crt
|   |-- apiserver.key
|   |-- ca.crt
|   |-- ca.key
|   |-- etcd
|   |   |-- ca.crt
|   |   |-- ca.key
|   |   |-- healthcheck-client.crt
|   |   |-- healthcheck-client.key
|   |   |-- peer.crt
|   |   |-- peer.key
|   |   |-- server.crt
|   |   `-- server.key
|   |-- front-proxy-ca.crt
|   |-- front-proxy-ca.key
|   |-- front-proxy-client.crt
|   |-- front-proxy-client.key
|   |-- sa.key
|   `-- sa.pub
`-- scheduler.conf
```

on a side note (to understand TLS / openssl commands )

https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-certs/

```

openssl genrsa -out ca.key 2048
openssl req -new -key ca.key -subj "/CN=K8s-ca" -out ca.csr
openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt
```

To view the certs 
```
openssl req -in ca.csr -noout -text
openssl x509 -in ca.crt -text -noout

```

To verify

```
openssl rsa -noout -modulus -in ca.key | openssl md5
openssl x509 -noout -modulus -in ca.crt | openssl md5
```

## Understand Static pods

```
root@kind-control-plane:/etc/kubernetes/manifests# ls -ltr
total 16
-rw------- 1 root root 3692 Nov 22 19:28 kube-apiserver.yaml
-rw------- 1 root root 3380 Nov 22 19:28 kube-controller-manager.yaml
-rw------- 1 root root 1384 Nov 22 19:28 kube-scheduler.yaml
-rw------- 1 root root 2115 Nov 22 19:28 etcd.yaml
```
***kube-apiserver***
```
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubeadm.kubernetes.io/kube-apiserver.advertise-address.endpoint: 192.168.48.5:6443
  creationTimestamp: null
  labels:
    component: kube-apiserver
    tier: control-plane
  name: kube-apiserver
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-apiserver
    - --advertise-address=192.168.48.5
    - --allow-privileged=true
    - --authorization-mode=Node,RBAC
    - --client-ca-file=/etc/kubernetes/pki/ca.crt
    - --enable-admission-plugins=NodeRestriction
    - --enable-bootstrap-token-auth=true
    - --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt
    - --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt
    - --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key
    - --etcd-servers=https://127.0.0.1:2379
    - --insecure-port=0
    - --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt
    - --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key
    - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
    - --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt
    - --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key
    - --requestheader-allowed-names=front-proxy-client
    - --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt
    - --requestheader-extra-headers-prefix=X-Remote-Extra-
    - --requestheader-group-headers=X-Remote-Group
    - --requestheader-username-headers=X-Remote-User
    - --runtime-config=
    - --secure-port=6443
    - --service-account-key-file=/etc/kubernetes/pki/sa.pub
    - --service-cluster-ip-range=10.96.0.0/16
    - --tls-cert-file=/etc/kubernetes/pki/apiserver.crt
    - --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
    image: k8s.gcr.io/kube-apiserver:v1.19.1
    imagePullPolicy: IfNotPresent
    livenessProbe:
      failureThreshold: 8
      httpGet:
        host: 192.168.48.5
        path: /livez
        port: 6443
        scheme: HTTPS
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    name: kube-apiserver
    readinessProbe:
      failureThreshold: 3
      httpGet:
        host: 192.168.48.5
        path: /readyz
        port: 6443
        scheme: HTTPS
      periodSeconds: 1
      timeoutSeconds: 15
    resources:
      requests:
        cpu: 250m
    startupProbe:
      failureThreshold: 24
      httpGet:
        host: 192.168.48.5
        path: /livez
        port: 6443
        scheme: HTTPS
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    volumeMounts:
    - mountPath: /etc/ssl/certs
      name: ca-certs
      readOnly: true
    - mountPath: /etc/ca-certificates
      name: etc-ca-certificates
      readOnly: true
    - mountPath: /etc/kubernetes/pki
      name: k8s-certs
      readOnly: true
    - mountPath: /usr/local/share/ca-certificates
      name: usr-local-share-ca-certificates
      readOnly: true
    - mountPath: /usr/share/ca-certificates
      name: usr-share-ca-certificates
      readOnly: true
  hostNetwork: true
  priorityClassName: system-node-critical
  volumes:
  - hostPath:
      path: /etc/ssl/certs
      type: DirectoryOrCreate
    name: ca-certs
  - hostPath:
      path: /etc/ca-certificates (Nothing there )
      type: DirectoryOrCreate
    name: etc-ca-certificates
  - hostPath:
      path: /etc/kubernetes/pki (Refer tree)
      type: DirectoryOrCreate
    name: k8s-certs
  - hostPath:
      path: /usr/local/share/ca-certificates (Nothing there)
      type: DirectoryOrCreate
    name: usr-local-share-ca-certificates
  - hostPath:
      path: /usr/share/ca-certificates (mozilla dir)
      type: DirectoryOrCreate
    name: usr-share-ca-certificates
status: {}
```

Pay attention to the volumes hostpath

*** Write more on kube-api server ***
( incoming traffics to kube-api server...)

## Kubelet (Worker Configuration)
From worker node perspective, you need to take a look at the following directory paths.
[For reference on kubeadm -> kubelet](#https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/kubelet-integration/#:~:text=When%20you%20call%20kubeadm%20init,)


1. /var/lib/
2. /etc/kubernetes/

/var/lib structure 
```
root@kind-worker:/var/lib# ls -ltr
total 48
drwxr-xr-x  2 root root 4096 Jul 27 19:44 misc
drwx------  2 root root 4096 Aug 26 21:58 private
drwxr-xr-x  4 root root 4096 Nov 22 19:20 nfs
drwxr-xr-x  2 root root 4096 Nov 22 19:20 pam
drwxr-xr-x  3 root root 4096 Nov 22 19:20 polkit-1
drwxr-xr-x  6 root root 4096 Nov 22 19:20 systemd
drwxr-xr-x  3 root root 4096 Nov 22 19:20 ucf
drwx--x--x 11 root root 4096 Nov 22 19:22 containerd
drwx------  8 root root 4096 Nov 22 19:41 kubelet
drwxr-xr-x  2 root root 4096 Nov 22 19:42 calico
drwxr-xr-x  5 root root 4096 Nov 23 16:33 apt
drwxr-xr-x  7 root root 4096 Nov 23 16:33 dpkg
```

tree structure of kubelet directory 

```
root@kind-worker:/var/lib/kubelet# tree
.
|-- config.yaml
|-- cpu_manager_state
|-- device-plugins
|   |-- DEPRECATION
|   |-- kubelet.sock
|   `-- kubelet_internal_checkpoint
|-- kubeadm-flags.env
|-- pki
|   |-- kubelet-client-2020-11-22-19-41-48.pem
|   |-- kubelet-client-current.pem -> /var/lib/kubelet/pki/kubelet-client-2020-11-22-19-41-48.pem
|   |-- kubelet.crt
|   `-- kubelet.key
|-- plugins
|-- plugins_registry
|-- pod-resources
|   `-- kubelet.sock
`-- pods
    |-- 149fdaea-00f3-41c7-b5c8-7c516033edfb
    |   |-- containers
    |   |   `-- kube-proxy
    |   |       `-- faca937b
    |   |-- etc-hosts
    |   |-- plugins
    |   |   `-- kubernetes.io~empty-dir
    |   |       |-- wrapped_kube-proxy
    |   |       |   `-- ready
    |   |       `-- wrapped_kube-proxy-token-b26hz
    |   |           `-- ready
    |   `-- volumes
    |       |-- kubernetes.io~configmap
    |       |   `-- kube-proxy
    |       |       |-- config.conf -> ..data/config.conf
    |       |       `-- kubeconfig.conf -> ..data/kubeconfig.conf
    |       `-- kubernetes.io~secret
    |           `-- kube-proxy-token-b26hz
    |               |-- ca.crt -> ..data/ca.crt
    |               |-- namespace -> ..data/namespace
    |               `-- token -> ..data/token
    |-- 227c1e21-c606-4b1b-bee7-8d48e9645892
    |   |-- containers
    |   |   `-- calico-typha
    |   |       `-- 2b71241b
    |   |-- etc-hosts
    |   |-- plugins
    |   |   `-- kubernetes.io~empty-dir
    |   |       |-- wrapped_calico-typha-token-ch4hb
    |   |       |   `-- ready
    |   |       |-- wrapped_typha-ca
    |   |       |   `-- ready
    |   |       `-- wrapped_typha-certs
    |   |           `-- ready
    |   `-- volumes
    |       |-- kubernetes.io~configmap
    |       |   `-- typha-ca
    |       |       `-- caBundle -> ..data/caBundle
    |       `-- kubernetes.io~secret
    |           |-- calico-typha-token-ch4hb
    |           |   |-- ca.crt -> ..data/ca.crt
    |           |   |-- namespace -> ..data/namespace
    |           |   `-- token -> ..data/token
    |           `-- typha-certs
    |               |-- cert.crt -> ..data/cert.crt
    |               |-- common-name -> ..data/common-name
    |               `-- key.key -> ..data/key.key
    `-- 67d1b7c9-4176-40cf-9ad1-f9a469655998
        |-- containers
        |   |-- calico-node
        |   |   `-- bdd9d192
        |   |-- flexvol-driver
        |   |   `-- ddc01942
        |   `-- install-cni
        |       `-- a7d83c4f
        |-- etc-hosts
        |-- plugins
        |   `-- kubernetes.io~empty-dir
        |       |-- wrapped_calico-node-token-cn8x2
        |       |   `-- ready
        |       |-- wrapped_felix-certs
        |       |   `-- ready
        |       `-- wrapped_typha-ca
        |           `-- ready
        `-- volumes
            |-- kubernetes.io~configmap
            |   `-- typha-ca
            |       `-- caBundle -> ..data/caBundle
            `-- kubernetes.io~secret
                |-- calico-node-token-cn8x2
                |   |-- ca.crt -> ..data/ca.crt
                |   |-- namespace -> ..data/namespace
                |   `-- token -> ..data/token
                `-- felix-certs
                    |-- cert.crt -> ..data/cert.crt
                    |-- common-name -> ..data/common-name
                    `-- key.key -> ..data/key.key

48 directories, 46 files
root@kind-worker:/var/lib/kubelet#
```

Here is snippet of config.yml

```
root@kind-worker:/var/lib/kubelet# cat config.yaml
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
    enabled: false
  webhook:
    cacheTTL: 0s
    enabled: true
  x509:
    clientCAFile: /etc/kubernetes/pki/ca.crt
authorization:
  mode: Webhook
  webhook:
    cacheAuthorizedTTL: 0s
    cacheUnauthorizedTTL: 0s
clusterDNS:
- 10.96.0.10
clusterDomain: cluster.local
cpuManagerReconcilePeriod: 0s
evictionHard:
  imagefs.available: 0%
  nodefs.available: 0%
  nodefs.inodesFree: 0%
evictionPressureTransitionPeriod: 0s
fileCheckFrequency: 0s
healthzBindAddress: 127.0.0.1
healthzPort: 10248
httpCheckFrequency: 0s
imageGCHighThresholdPercent: 100
imageMinimumGCAge: 0s
kind: KubeletConfiguration
logging: {}
nodeStatusReportFrequency: 0s
nodeStatusUpdateFrequency: 0s
rotateCertificates: true
runtimeRequestTimeout: 0s
staticPodPath: /etc/kubernetes/manifests
streamingConnectionIdleTimeout: 0s
syncFrequency: 0s
volumeStatsAggPeriod: 0s
root@kind-worker:/var/lib/kubelet#
```

Question: How does the kube-proxy gets initialized?. Where do i see the details of the spec ?

***Important***
To validate the certs with chain

```
openssl verify -CAfile <cafile.pem> -untrusted serverchain.pem servercert.pem; openssl rsa -noout -modulus -in serverkey.pem | openssl x509 -in servercert.pem -modulus -noout | openssl md5; 
```

openssl verify -CAfile pki/ca.crt -untrusted /var/lib/kubelet/pki/kubelet-client-current.crt ; openssl x509 -noout -modulus -in /var/lib/kubelet/pki/kubelet-client-current.pem | openssl md5 

openssl rsa -noout -modulus -in /etc/kubernetes/pki/ca.crt | openssl md5

/var/lib/kubelet/pki/kubelet-client-current.pem

```
openssl rsa -noout -modulus -in ca.key | openssl md5
openssl x509 -noout -modulus -in ca.crt | openssl md5
```

/var/lib/kubelet/pki/kubelet-client-current.pem
/var/lib/kubelet/pki/kubelet-client-current.pem

## Kube-Proxy (Kind )

This runs as a dameonset 
Needs to be confirmed with kube-adm (In aws installation..)

```
# cat /var/lib/kube-proxy/config.conf
apiVersion: kubeproxy.config.k8s.io/v1alpha1
bindAddress: 0.0.0.0
bindAddressHardFail: false
clientConnection:
  acceptContentTypes: ""
  burst: 0
  contentType: ""
  kubeconfig: /var/lib/kube-proxy/kubeconfig.conf
  qps: 0
clusterCIDR: 10.244.0.0/16
configSyncPeriod: 0s
conntrack:
  maxPerCore: null
  min: null
  tcpCloseWaitTimeout: null
  tcpEstablishedTimeout: null
detectLocalMode: ""
enableProfiling: false
healthzBindAddress: ""
hostnameOverride: ""
iptables:
  masqueradeAll: false
  masqueradeBit: null
  minSyncPeriod: 1s
  syncPeriod: 0s
ipvs:
  excludeCIDRs: null
  minSyncPeriod: 0s
  scheduler: ""
  strictARP: false
  syncPeriod: 0s
  tcpFinTimeout: 0s
  tcpTimeout: 0s
  udpTimeout: 0s
kind: KubeProxyConfiguration
metricsBindAddress: ""
mode: iptables
nodePortAddresses: null
oomScoreAdj: null
portRange: ""
showHiddenMetricsForVersion: ""
udpIdleTimeout: 0s
winkernel:
  enableDSR: false
  networkName: ""
  sourceVip: ""# cat /var/lib/kube-proxy/kubeconfig.conf
apiVersion: v1
kind: Config
clusters:
- cluster:
    certificate-authority: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
    server: https://kind-control-plane:6443
  name: default
contexts:
- context:
    cluster: default
    namespace: default
    user: default
  name: default
current-context: default
users:
- name: default
  user:
    tokenFile: /var/run/secrets/kubernetes.io/serviceaccount/token#
#
#
# cat /var/lib/kube-proxy/kubeconfig.conf
apiVersion: v1
kind: Config
clusters:
- cluster:
    certificate-authority: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
    server: https://kind-control-plane:6443
  name: default
contexts:
- context:
    cluster: default
    namespace: default
    user: default
  name: default
current-context: default
users:
- name: default
  user:
    tokenFile: /var/run/secrets/kubernetes.io/serviceaccount/token# openssl
sh: 9: openssl: not found
```

