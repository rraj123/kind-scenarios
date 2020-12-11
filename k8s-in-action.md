# K8s-in-action 
Reading notes and code snippet 

#### Chapter 4  <br>

***Kubernetes API***

Standard RESTful APIs <br>

`POST`, `GET` ,`PUT` ,`PATCH` ,`DELETE` <br>

Resource and object means the same thing.  <br>

An single object instance can also be exposed through more than one REST resource (URI).

In RDBMS world, you can compare resource and objects as tables and views. 

In k8s, resource could also mean cpu, mem etc. 

So, use objects. 


Resource (API) -> Object (server)

#### Object Manifests

+ Type Metadata `apiVersion`,`kind`
+ Object Metadata
 ```
 metadata:
   name:
   labeles:
 ``` 
+ Spec (Desired state)

+ Status (Actual state)

Controller updates the state. 

Node objects are created by kubelets(They are slightly different)

How do you access the APIs directly?. 

`kubectl proxy`

`http://127.0.0.1:8001/api/v1/nodes/kind-control-plane`

`kubectl run --image=tutum/curl -it --restart=Never --rm client-pod`

`kubectl port-forward pod-name port-number`


#### Viewing Logs
```
/var/log/containers
docker logs `container-id` 
```
Log into the node that hosts the pod 
```
docker logs

```

Log streaming
```
kubectl logs
kubectl logs -f kubia --timestamps=true
```

Log Querying

```
kubectl logs -f kubia --timestamps=true
kubectl logs kubia --since=2m
kubectl logs kubia --since-time=2020-02-01T09:50:00Z
kubectl logs kubia --tail=10
```

The logs are stored under `/var/log/containers` on the node that runs the container. Separate file is created for each container, If the containers are restarted then the new file will be created. 

The kubectl logs -f will get killed. 

To view the previous container, `--previous` `-p` option. 

So, it is ideal to set up a cluster wide logging system to make those logs available for 

***Application Logs*** <br>
If you app writes to stdout, instead of stdout, you will need to have another service to scrap the logs. 

***Copying files to and from Containers***
```
kubectl cp kubia:/etc/hosts /tmp/kubia-hosts
```
local to container
```
kubectl cp /path/to/local/file kubia:path/in/container
```
The kubectl cp command requires the tar binary to be present in your container, but this requirement may change in the future

***Execute command in container***
```
kubectl exec pod -- ps aux
kubectl exec kubia -- curl -s localhost:8080
```

Interactive 
```
kubectl exec -it kubia -- bash

```

***Attaching to a running container***
`kubectl attach`

***Running multiple containers***


***Init Containers***

***Pod Phases***

```
kubectl get po kubia -o yaml | grep phase
kubectl get po kubia -o json | jq .status.phase
```

<u>Container State </u>

```
kubectl get po kubia -o json | jq .status.containerStatuses
kubectl get po kubia-init -o json | jq .status 
```
For Init containers the status of the field is available through

`status` section of the pod object manifest, but in the `initContainerStatuses`

```
$ kubectl apply -f kubia-ssl.yaml
$ kubectl port-forward kubia-ssl 8080 8443 9901
```

To terminate the envoy container , you need to open the `envoy` container 
```
curl -X POST http://localhost:9901/quitquitquit
```
<u>Pod Restart </u>

Here are the restart policy state model

```
restartPolicy: Always
restartPolicy: OnFailure (For Non-zero exit code)
restartPolicy: Never

```

The time dealy is inserted before a container is restarted. 

<u>Liveness probe (To check container health)</u>

Kubernetes runs the probe for each containers in the pod. 
1. Http Get
```
    livenessProbe:
      httpGet:
        path: /
        port: 8080
---
curl -X POST localhost:9901/healthcheck/fail
curl -X POST localhost:9901/healthcheck/ok
```
2. TCP Socket
```

    livenessProbe:
      tcpSocket:
        port: 1234
      periodSeconds: 2
      failureThreshold: 1
```
3. Exec Probe. (Command inside the container and checks the exit code and it terminates with.)
```

    livenessProbe:
      exec:
        command:
        - /usr/bin/healthcheck
      periodSeconds: 2
      timeoutSeconds: 1
      failureThreshold: 1
```

Example
```
apiVersion: v1
kind: Pod
metadata:
  name: kubia-liveness
spec:
  containers:
  - name: kubia
    image: luksa/kubia:1.0
    ports:
    - name: http
      containerPort: 8080
    livenessProbe:
      httpGet:
        path: /
        port: 8080
  - name: envoy
    image: luksa/kubia-ssl-proxy:1.0
    ports:
    - name: https
      containerPort: 8443
    - name: admin
      containerPort: 9901
    livenessProbe:
      httpGet:
        path: /ready
        port: admin
      initialDelaySeconds: 10
      periodSeconds: 5
      timeoutSeconds: 2
      failureThreshold: 3
```


```
$ kubectl logs kubia-liveness -c kubia -f
$ kubectl exec kubia-liveness -c envoy -- tail -f /var/log/envoy.admin.log
$ curl -X POST localhost:9901/healthcheck/fail
kubectl logs kubia-liveness -c envoy -p

```

<u>Startup Probe </u>
The startup probe is executed when the container is started. The startup probe can be configured to take into account the slow start of the application. 

When the startup probe succeeds, Kubernetes switches to using the liveness probe, which is configured to quickly detect when the application becomes unhealthy.

```
  containers:
  - name: kubia
    image: luksa/kubia:1.0
    ports:
    - name: http
      containerPort: 8080
    startupProbe:
      httpGet:
        path: /
        port: http
      periodSeconds: 10
      failureThreshold:  12
    livenessProbe:
      httpGet:
        path: /
        port: http
      periodSeconds: 5
      failureThreshold: 2
```

Usecase: <br>
When the container is slow to start, then you can have startup to check the status of the container.

The above conf gives 120 seconds to start the container.
If startup probe does not start 

***Tip*** <br>

If a poorly implemented probe returns a negative response even though the application is healthy, the application will be restarted unnecessarily. <br>
`/healthz` -> checks the internal 

If many services are interdependent in this way, the failure of a single service can result in cascading failures across the entire system.


#### <u>Executing actions at container start-up and shutdown </u>

`Post-start` <br>
`pre-stop`

+ Execute a command inside the container, 
+ Send an HTTP GET request to the application in the container

***Tip*** <br>
Lifecycle hooks apply to containers and not to pods

??? What happens if you have a multiple containers in the same pod and one of the container is continue to fail ???

### <u> Pod Lifecycle </u>

1. Init Stage (Pod's containers init containers run)
2. Run Stage ( Regular containers of the pod run)
3. Termination Stage ( Pod containers are terminated)

#### <u> Init Stage </u>
Init Containers are executed only once, Even if one of the pods main container is terminated later, the pods init containers are not executed re-executed. 

Containers don’t necessarily start at the same moment. 

Init container must be idempotent. 

A long-running pre-stop hook does block the shutdown of the container in which it is defined, but it does not block the shutdown of other containers.

<u>ImagePullPolicy</u>

+ Always 
+ Never
+ IfNotPresent (default)

If the imagePullPolicy is set to Always and the image registry is offline, the container will not run even if the same image is already stored locally.

<u>Pod's RestartPolicy</u>

+ Always 
+ OnFailure 
+ Never (Goes to error and must be deleted and recreated)

If the container needs to be restarted and imagePullPolicy is set to Always, the container image is pulled again.

 <i><Init Containers are executed only once, Even if one of the pods main container is terminated later, the pods init containers are not executed re-executed.></i><>

#### <u> Run Stage </u>
When all init containers are successfully completed, the pod’s regular containers are all created in parallel.

The Kubelet doesn’t start all containers of the pod at the same time.

The `post-start hook` runs asynchronously with the main container process. 

The termination of containers is performed in parallel.

`pre-stop` hook does block the shutdown of other containers. 

??? If the container is in error, how many times the pod tries to restart. ???

<u>`terminationGracePeriodSeconds`</u>
The timer starts when the pre-stop hook is called or when the TERM signal is sent if no hook is defined. If the process is still running after the termination grace period has expired, it’s terminated by force via the KILL signal. This terminates the container.

#### <u>Understanding the termination stage</u>

`deletionGracePeriodSeconds`

`metadata.deletionGracePeriodSeconds` defined in pod's metadata

By default, it gets its value from the spec.terminationGracePeriodSeconds field, but you can specify a different value in the kubectl delete command.

## <u>K8s Volumes </u>
<br>


Mounting is the act of attaching the filesystem of some storage device or volume into a specific location in the operating system’s file tree.

The lifecylcle of a volume is tied to the lifecycle of the entire pod and independent of the container in which it is mounted. 

#### <b> Persisting files across container restarts </b>

All volumes in a pod are created when the pod is set up - before any of its containers are started

First, you need to determine what data needs to be retained. 

Mounting multiple volumes in a container. 

Pods with multiple containers <br>
Sharing files between multiple containers. <br>
 (Side car type of scenarios)

Volume mount can be either `ro` or `r/w`

Single volume in two containers are cases where <u>a sidecar container runs a tool that processes or rotates the web server logs or when an init container creates configuration files for the main application container.</u>


#### <b> Persisting data across pod restarts </b>

With NAS , same storage volume is attached to the pod. 

#### <b> Sharing data between the pods </b>
Simple case: local directory

If the persistent storage is a <u>network-attached storage volume</u>, the pods may be able to use across the nodes.

`configMap, secret, downwardAPI, and the projected` are type—Special types of volumes used to expose information about the pod and other Kubernetes objects through files.

When you share volume between containers, there could be short period of time... where... (so use Init Container to .. )

<u>Access Mode</u> <br>

`ReadWriteOnce, ReadOnlyMany, ReadWriteMany`

<u>reclaim policy on persistent volumes </u>

`Retain,Recycle,Delete` 

#### <u> Dynamic provisioning of persistent volumes </u>

#### StorageClass object 
+ Standard, 
+ Fast 
+ Nfs


