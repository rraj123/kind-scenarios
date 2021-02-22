## ETCD

### Introduction

you can get the more info using


```
kubectl get pods -n kube-system
```

Install etcdctl client tool. 

```
etcdctl get / --prefix -keys-only
```

(Optional) Additional information about ETCDCTL Utility

ETCDCTL is the CLI tool used to interact with ETCD.

ETCDCTL can interact with ETCD Server using 2 API versions - Version 2 and Version 3.  By default its set to use Version 2. Each version has different sets of commands.

For example ETCDCTL version 2 supports the following commands:

    etcdctl backup
    etcdctl cluster-health
    etcdctl mk
    etcdctl mkdir
    etcdctl set


Whereas the commands are different in version 3

    etcdctl snapshot save 
    etcdctl endpoint health
    etcdctl get
    etcdctl put


To set the right version of API set the environment variable ETCDCTL_API command

export ETCDCTL_API=3


When API version is not set, it is assumed to be set to version 2. And version 3 commands listed above don't work. When API version is set to version 3, version 2 commands listed above don't work.


Apart from that, you must also specify path to certificate files so that ETCDCTL can authenticate to the ETCD API Server. The certificate files are available in the etcd-master at the following path. We discuss more about certificates in the security section of this course. So don't worry if this looks complex:

    --cacert /etc/kubernetes/pki/etcd/ca.crt     
    --cert /etc/kubernetes/pki/etcd/server.crt     
    --key /etc/kubernetes/pki/etcd/server.key


So for the commands I showed in the previous video to work you must specify the ETCDCTL API version and path to certificate files. Below is the final form:

The following command fetches the first 100 lines from etcd 

```
kubectl exec etcd-kind-control-plane -n kube-system -- sh -c "ETCDCTL_API=3 etcdctl get / --prefix --keys-only --limit=100 --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/server.crt  --key /etc/kubernetes/pki/etcd/server.key"   
```



------------------------------------------------------------------------------------------


## Revise


```

export ETCDCTL_API=3


ETCDCTL_API=3 etcdctl member list /opt/snapshot-pre-boot.db --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/apiserver-etcd-client.crt --key=/etc/kubernetes/pki/apiserver-etcd-client.key

o/p:
173cb292082c8c5a, started, controlplane, https://172.17.0.34:2380, https://172.17.0.34:2379, false


The following one without the endpoint (It works)

ETCDCTL_API=3 etcdctl snapshot save /opt/snapshot-pre-boot.db --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/apiserver-etcd-client.crt --key=/etc/kubernetes/pki/apiserver-etcd-client.key 


The following one with the endpoint - from the member list .. (works too..)
ETCDCTL_API=3 etcdctl snapshot save /opt/snapshot-pre-boot.db --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/apiserver-etcd-client.crt --key=/etc/kubernetes/pki/apiserver-etcd-client.key --endpoint=127.0.0.1:2379

-----

memebers list 

ETCDCTL_API=3 etcdctl member list /opt/snapshot-pre-boot.db --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/apiserver-etcd-client.crt --key=/etc/kubernetes/pki/apiserver-etcd-client.key
o/p:
4fa49499e903596a, started, controlplane, https://172.17.0.17:2380, https://172.17.0.17:2379, false

---
ETCDCTL_API=3 etcdctl snapshot restore -h

ETCDCTL_API=3 etcdctl snapshot restore /opt/snapshot-pre-boot.db --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/apiserver-etcd-client.crt --key=/etc/kubernetes/pki/apiserver-etcd-client.key --data-dir="/var/lib/etcd-from-backup1" --initial-cluster="controlplane=https://172.17.0.34:2380" --name="controlplane" --initial-advertise-peer-urls="https://172.17.0.34:2380" --initial-cluster-token="etcd-cluster-1" 




ontrolplane $ ETCDCTL_API=3 etcdctl snapshot restore /opt/snapshot-pre-boot.db --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/apiserver-etcd-client.crt --key=/etc/kubernetes/pki/apiserver-etcd-client.key --data-dir="/var/lib/etcd-from-backup1" --initial-cluster="controlplane=https://127.0.0.1:2380" --name="controlplane" --initial-advertise-peer-urls="https://127.0.0.1:2380" --initial-cluster-token="etcd-cluster-2"

{"level":"info","ts":1607755195.8343222,"caller":"snapshot/v3_snapshot.go:296","msg":"restoring snapshot","path":"/opt/snapshot-pre-boot.db","wal-dir":"/var/lib/etcd-from-backup1/member/wal","data-dir":"/var/lib/etcd-from-backup1","snap-dir":"/var/lib/etcd-from-backup1/member/snap"}

{"level":"info","ts":1607755195.8726413,"caller":"mvcc/kvstore.go:380","msg":"restored last compact revision","meta-bucket-name":"meta","meta-bucket-name-key":"finishedCompactRev","restored-compact-revision":5376}

{"level":"info","ts":1607755195.8935237,"caller":"membership/cluster.go:392","msg":"added member","cluster-id":"17b9ca8f6358219d","local-member-id":"0","added-peer-id":"3d7bbb806e1acbd5","added-peer-peer-urls":["https://127.0.0.1:2380"]}

{"level":"info","ts":1607755195.9398437,"caller":"snapshot/v3_snapshot.go:309","msg":"restored snapshot","path":"/opt/snapshot-pre-boot.db","wal-dir":"/var/lib/etcd-from-backup1/member/wal","data-dir":"/var/lib/etcd-from-backup1","snap-dir":"/var/lib/etcd-from-backup1/member/snap"}
controlplane $



ETCDCTL_API=3 etcdctl snapshot restore /opt/snapshot-pre-boot.db  --data-dir="/var/lib/etcd-from-backup2"  --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/apiserver-etcd-client.crt --key=/etc/kubernetes/pki/apiserver-etcd-client.key

ETCDCTL_API=3 etcdctl snapshot restore /opt/snapshot-pre-boot.db --data-dir /var/lib/etcd-backup

```
