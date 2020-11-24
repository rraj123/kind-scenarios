## Open Question

1. kube-api server location (service details, look at the entire structure and understand other configuration... ) 
   (In Kubeadm /etc/kubernetes/manifests/kube-apiserver.yaml)
    ( cat /etc/systemd/system/kube-apiserver.service)

2. Kubelet is not managed by the kubeadm tool.
   (kubelet has to be manually downloaded and installed.)

   to view the kubelet process
   ```
   ps -aux | grep kubelet
   ```
Note: What are the static ... 
```
-rw-rw-r-- 1 root root 1013 Aug 26 21:22 10-kubeadm.conf
root@kind-control-plane:/etc/systemd/system/kubelet.service.d# cat 10-kubeadm.conf
# https://github.com/kubernetes/kubernetes/blob/ba8fcafaf8c502a454acd86b728c857932555315/build/debs/10-kubeadm.conf
# Note: This dropin only works with kubeadm and kubelet v1.11+
[Service]
Environment="KUBELET_KUBECONFIG_ARGS=--bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf"
Environment="KUBELET_CONFIG_ARGS=--config=/var/lib/kubelet/config.yaml"
# This is a file that "kubeadm init" and "kubeadm join" generates at runtime, populating the KUBELET_KUBEADM_ARGS variable dynamically
EnvironmentFile=-/var/lib/kubelet/kubeadm-flags.env
# This is a file that the user can use for overrides of the kubelet args as a last resort. Preferably, the user should use
# the .NodeRegistration.KubeletExtraArgs object in the configuration files instead. KUBELET_EXTRA_ARGS should be sourced from this file.
EnvironmentFile=-/etc/default/kubelet
ExecStart=
ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTRA_ARGSroot@kind-control-plane:/etc/systemd/system/kubelet.service.d#

```


3. kube-proxy
It is not a container
Kube-proxy is an IP tables rules 
In kube-adm deploys kube-proxy as pods (Daemonsets)

4. 


