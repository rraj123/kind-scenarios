# K8s Troubleshooting

#### App Failure

```
kubectl get pod
kubectl describe pod web
kubectl logs web -f --previous

```

refer link ??? troubleshooting applications. 
```
This should display the endpoint for the services if they are already bound to it. 
kubectl get ep 

```

#### Control plane

How do you findout or confirm location of the static paths?. <br>
To be able to debug control plane components, looks for kubelet 

`cat /etc/systemd/system/kubelet.service.d/10.kubeadm.conf`
look for `KUBELET_CONFIG_ARGS=--config=/var/lib/kubelet/config.yaml`

Then search for the `staticPodPath` in the config.yaml file 

From this point onwards you can find the static pod path, and look at the configuration files. 

`kubectl scale deploy app --replicas=2`

#### Worker node

So, what is the theme here?

1. kubelet service (systemctl or cat `cat /etc/systemd/system/kubelet.service.d/) -> config=/var/lib/kubelet/config.yaml`
2. authentication cert
3. wrong info to connect to different services. 
4. where do you find error information .. `journalctl -u kubelet`
5. 
`kubectl cluster-info`
use these commands

```
journalctl -u kubelet (shift+g to move forward with the messages cntrl+f or cntrl+b)

```

#### Network
The kube-proxy pods are not running. As a result the rules needed to allow connectivity  to the services have not been created.

1. Check the logs of the kube-proxy pods
   kubectl -n kube-system logs <name_of_the_kube_proxy_pod>

2. The configuration file /var/lib/kube-proxy/configuration.conf is not valid. The configuration path does not match the data in the ConfigMap.
   kubectl -n kube-system describe configmap kube-proxy shows that the file name used is  config.conf which is mounted in the kube-proxy damonset podsat the path /var/lib/kube-proxy/config.conf 
   /var/lib/kube-proxy/kubeconfig.conf 
   /var/lib/kube-proxy/kubeconfig.conf
3. However in the DaemonSet for kube-proxy, the command used to start the kube-proxy pod makes use of the path /var/lib/kube-proxy/configuration.conf.

  Correct this path to /var/lib/kube-proxy/config.conf as per the ConfigMap and recreate the kube-proxy pods.

This should get the kube-proxy pods back in a running state.


***Error***
```
failed to decode: no kind "Config" is registered for version "v1" in scheme 

```

```
The kube-dns service is not working as expected. The first thing to check is if the service has a valid endpoint? Does it point to the kube-dbs/core-dns ?

Run: kubectl -n kube-system get ep kube-dns

If there are no endpoints for the service, inspect the service and make sure it uses the correct selectors and ports.

Run: kubectl -n kube-system descrive svc kube-dns

Note that the selector used is: k8s-app=core-dns

If you compare this with the label set on the coredns deployment and its pods, you will see that the select should be k8s-app=kube-dns

Modify the kube-dns service and update the selector to k8s-app=kube-dns
(Easist way is to use the kubectl edit command)
```