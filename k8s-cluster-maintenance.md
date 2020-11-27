## K8S- Cluster Maintenance

#### OS Upgrades

When the node goes to maintenance for a period of time, then the master node puts them on .. 



kubecontroller Manager manages the pod-eviction-timeout 

drain the node then uncordon the node. 

`kubectl drain node` <br>
`kubectl uncordon node` -> to place the pods again on these nodes. 

`kubectl cordon node` -> to ensure that the new pods are not going to be placed on this node. 


#### Cluster upgrade process

??? what is the ignore will do?

you are moving the pod out of the node
```
kubectl drain node01 --ignore-daemonsets
```

make the node schedulable again. 
```
kubectl uncordon node01
```

if ther are any loose pods with the nodes, then you cannot drain the node. you need to use `--force` switch. 

***The loose pods will be deleted forever*** 

<u>To make it not schedulable</u> : 
`kubectl cordon node03`


Releases
```
https://kubernetes.io/docs/concepts/overview/kubernetes-api/

Here is a link to kubernetes documentation if you want to learn more about this topic (You don't need it for the exam though):

https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/api-conventions.md

https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/api_changes.md
```

#### Upgrade Process

To upgrade K8s, first find out the version of the kube-apiserver, then the rest of components can be two versions lower except kubectl. 

coredns, etcd are comes in different bundle. 

you cannot upgrade from v1.10 to v1.12 directly, you will have to go one minor version at a time. That is the recommended approach. 

<br>
`kubeadm upgrade plan` <br>
`kubeadm upgrade apply`

1. Upgrade all nodes
2. Upgrade one at a time
3. Buy excessive capacity

Remember, during the master upgrade process , the control plane will not available for management functions. The worker nodes will continue to work, however, the rs, dep (k8s) wont work. 

With kubeadm, you can use the above commands to perform cluster operations. The `kubelet` has to be upgraded manually.  

`kubeadm` also has to be updated. 

`yum install -y kubeadm=1.12.` <br> then apply
`kubeadm upgrade apply v1.15.24`

??? Does kubelet gets installed on the master node ???

Pay attention to the output. 

kubelet runs on all the nodes.. 

`apt-get upgrade -y kubelet=ccc` <br>
`systemctl restart kubelet` <br>
`kubectl get nodes`

#### Upgrade

Here is the sequence
```
kubectl drain node
apt-get upgrade -y kubeadm=1...
apt-get upgrade -y kubelet=1...
kubeadm upgrade node config --kubelet-version v1.12.4
systemctl restart kubelet
kubectl uncordon node
```


***Practice Tips***

To upgrade master node

This is to make the node 
```
kubectl drain master/controlplane --ignore-daemonsets
```

you need upgrade the kubeadm tool itself

??? How do you know different versions of available kubeadm tool ???

```
apt update 
apt install kubeadm=1.19.0-00 
kubeadm upgrade apply v1.19.0  
apt install kubelet=1.19.0-00
```

make the master schedulable again
```
kubectl uncordon controlplane
```

Then next perform the same operation on the node01

```
kubectl drain node01 --ignore-daemonsets
```


```
apt update 
apt install kubeadm=1.19.0-00 
kubeadm upgrade node
apt install kubelet=1.19.0-00
```

Make the node schedulable again. 

```
kubectl uncordon node01
```

### Backup and Restore Methods

All the resources can be backup by 

`kubectl get all --all-namespaces -o yaml > all-deploy-services.yaml`

Backup etcd

First you need to find out the how etcd is configured and where is the data directory is stored. `--data-dir`

```
ETCDCTL_API=3 etcdctl snapshot save snapshot.db
ETCDCTL_API=3 etcdctl snapshot status snapshot.db

```
stop the kube-apiserver

```
service kube-apiserver stop
```

reload the 

#### ETCD Notes

```
etcdctl is a command line client for etcd.


In all our Kubernetes Hands-on labs, the ETCD key-value database is deployed as a static pod on the master. The version used is v3.

To make use of etcdctl for tasks such as back up and restore, make sure that you set the ETCDCTL_API to 3.


You can do this by exporting the variable ETCDCTL_API prior to using the etcdctl client. This can be done as follows:

export ETCDCTL_API=3

On the Master Node:


To see all the options for a specific sub-command, make use of the -h or --help flag.


For example, if you want to take a snapshot of etcd, use:

etcdctl snapshot save -h and keep a note of the mandatory global options.


Since our ETCD database is TLS-Enabled, the following options are mandatory:

--cacert                                                verify certificates of TLS-enabled secure servers using this CA bundle

--cert                                                    identify secure client using this TLS certificate file

--endpoints=[127.0.0.1:2379]          This is the default as ETCD is running on master node and exposed on localhost 2379.

--key                                                      identify secure client using this TLS key file



Similarly use the help option for snapshot restore to see all available options for restoring the backup.

etcdctl snapshot restore -h

For a detailed explanation on how to make use of the etcdctl command line tool and work with the -h flags, check out the solution video for the Backup and Restore Lab.
```


#### ETCD - Practice

Where is the etcd is running?

```
look for the --listen-client-urls=https://12.0.0.1:2379
```

Reference :
https://github.com/mmumshad/kubernetes-the-hard-way/blob/master/practice-questions-answers/cluster-maintenance/backup-etcd/etcd-backup-and-restore.md

Try 
`ETCDCTL_API=3 etcdctl member -h`
`ETCDCTL_API=3 etcdctl member list -h`
`ETCDCTL_API=3 etcdctl snapshot restore -h`
Back up
```
ETCDCTL_API=3 

etcdctl  member list --cacert=/etc/kubernetes/pki/etcd/ca.crt \
    --cert=/etc/kubernetes/pki/etcd/server.crt \
    --key=/etc/kubernetes/pki/etcd/server.key \
    --endpoints=https://127.0.0.1:2379

d5f1962ed3267fa0, started, controlplane, https://172.17.0.53:2380, https://172.17.0.53:2379, false

ETCDCTL_API=3  etcdctl  snapshot save --cacert=/etc/kubernetes/pki/etcd/ca.crt \
    --cert=/etc/kubernetes/pki/etcd/server.crt \
    --key=/etc/kubernetes/pki/etcd/server.key \
    --endpoints=https://127.0.0.1:2379 \
    /opt/snapshot-pre-boot.db

```

```
      etcd
      --advertise-client-urls=https://172.17.0.16:2379
      --cert-file=/etc/kubernetes/pki/etcd/server.crt
      --client-cert-auth=true
      --data-dir=/var/lib/etcd
      --initial-advertise-peer-urls=https://172.17.0.16:2380
      --initial-cluster=controlplane=https://172.17.0.16:2380
      --key-file=/etc/kubernetes/pki/etcd/server.key
      --listen-client-urls=https://127.0.0.1:2379,https://172.17.0.16:2379
      --listen-metrics-urls=http://127.0.0.1:2381
      --listen-peer-urls=https://172.17.0.16:2380
      --name=controlplane
      --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt
      --peer-client-cert-auth=true
      --peer-key-file=/etc/kubernetes/pki/etcd/peer.key
      --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
      --snapshot-count=10000
      --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
```

```
Restore 
```
ETCDCTL_API=3 
etcdctl  --endpoints=127.0.0.1:2379  \
     --cacert=/etc/kubernetes/pki/etcd/ca.crt \
     --cert=/etc/kubernetes/pki/etcd/server.crt \
     --key=/etc/kubernetes/pki/etcd/server.key \
     --data-dir="/var/lib/etcd-from-backup" \
     --initial-cluster-token="etcd-cluster-1" \
     --name="controlplane"
     --initial-advertise-peer-urls="https://127.0.0.1:2380" \
     --initial-cluster="controlplane=https://127.0.0.1:2380" \
     snapshot restore /opt/snapshot-pre-boot.db
```

Output:
```
controlplane $ etcdctl  --endpoints=127.0.0.1:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key --data-dir="/var/lib/etcd-from-backup" --initial-cluster-token="etcd-cluster-1" --name="controlplane" --initial-advertise-peer-urls="https://127.0.0.1:2380" --initial-cluster="controlplane=https://127.0.0.1:2380" snapshot restore /opt/snapshot-pre-boot.db
{"level":"info","ts":1606404913.4199886,"caller":"snapshot/v3_snapshot.go:296","msg":"restoring snapshot","path":"/opt/snapshot-pre-boot.db","wal-dir":"/var/lib/etcd-from-backup/member/wal","data-dir":"/var/lib/etcd-from-backup","snap-dir":"/var/lib/etcd-from-backup/member/snap"}
{"level":"info","ts":1606404913.976208,"caller":"mvcc/kvstore.go:380","msg":"restored last compact revision","meta-bucket-name":"meta","meta-bucket-name-key":"finishedCompactRev","restored-compact-revision":1943}
{"level":"info","ts":1606404914.050492,"caller":"membership/cluster.go:392","msg":"added member","cluster-id":"7581d6eb2d25405b","local-member-id":"0","added-peer-id":"e92d66acd89ecf29","added-peer-peer-urls":["https://127.0.0.1:2380"]}
{"level":"info","ts":1606404914.5000327,"caller":"snapshot/v3_snapshot.go:309","msg":"restored snapshot","path":"/opt/snapshot-pre-boot.db","wal-dir":"/var/lib/etcd-from-backup/member/wal","data-dir":"/var/lib/etcd-from-backup","snap-dir":"/var/lib/etcd-from-backup/member/snap"}
controlplane $
```


The next step is to 

1. Validate by going to the restored directory exists `cd /var/lib/etcd-from-backup` {you should see memeber.. }
2. update the static pod with 
     a. update new directory (data path)
     b. Initial cluster token
     c. Mount path (Volume)

```
Update the data directory path
- --data-dir-path=
- --initial-cluster-token=

Volume mounts

volumeMounts:
- mountPath: 


volumes:
  - hostPath:
      path:


```


`docker ps -a | grep etcd`

you can verify them with the `docker ps` or `kubectl `

run member list command again you should see the new member list command. 


```
controlplane $ etcdctl  member list --cacert=/etc/kubernetes/pki/etcd/ca.crt \
>     --cert=/etc/kubernetes/pki/etcd/server.crt \
>     --key=/etc/kubernetes/pki/etcd/server.key \
>     --endpoints=https://127.0.0.1:2379
e92d66acd89ecf29, started, controlplane, https://127.0.0.1:2380, https://172.17.0.53:2379, false
controlplane $
```

Other References 
--------------

https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster

https://github.com/etcd-io/etcd/blob/master/Documentation/op-guide/recovery.md


https://www.youtube.com/watch?v=qRPNuT080Hk

***Always check with kubectl describe pod*** 


