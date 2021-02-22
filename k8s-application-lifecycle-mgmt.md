# k8s - Application Lifecyle Management

### Rollout and versioning

```
kubectl rollout status deployment
kubectl rollout history deployment/myapp-deployment
```

***Deployment Strategy***

1. Recreate
2. Rolling update

How do you update..

```
kubectl apply -f 
kubectl set image deployment/myapp nginx=nginx:1.9.1
```

Rollout <br>
`kubectl rollout undo deployment/myapp-deployment`

Watch out replicaset during the deployment update 

Some commands:

```
kubectl create -f deployment depl-file.yaml
kubectl get deployments
kubectl apply -f depl-file.yaml
kubectl set image deployment/myapp nginx=nginx:1.9.3
kubectl rollout status deployment/myapp
kubectl rollout history deployment/myapp
kubectl rollout undo deployment/myapp
```

### Environment variable

`docker run -e APP_CONN=localhost simple-app-conn`

using env value

```

spec:
  containers:
    - name:
     
      env:
    
```

1. ConfigMap
2. Secret

#### ConfigMap

create config map
Inject config map to pod sepc

Imperative to create ConfigMap

```
kubectl create configmap <config-name> --from-literal=<key>=<value>
kubectl create configmap app-config --from-literal=APP_COLOR=blue
-or-
kubectl create configmap <config-name> --from-from-file=
kubectl create configmap app-config --from-file=app_config.properties
```

ConfigMap in pods

1. env
2. volume 


??? What is difference between command vs args in pod definition

*** https://kubernetes.io/docs/tasks/inject-data-application/define-command-argument-container/ ***


#### Practice Env 

<b>Command and ars... Manually edit it .. </b>

```
apiVersion: v1
kind: Pod
metadata:
  name: webapp-green
  labels:
      name: webapp-green
spec:
  containers:
  - name: simple-webapp
    image: kodekloud/webapp-color
    args: ["--color", "green"] 

```
### Secrets 

Imperative
```
kubectl create secret generic <sec-name> --from-literal=<key>=<value>

kubectl create secret generic app-secret --from-literal=DB=mysql


kubectl create secret generic <sec-name> --from-file=file
```


```
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
data:
  DB_Host: mysql

To Encode
echo -n 'word' | base64
echo -n 'root' | base64
echo -n 'word' | base64 --decode
echo -n 'root' | base64 --decode
```

```
kubectl get secrets

```

you can inject the whole secret as volumes..
Then the secrets are created as volume inside the container... 

```

```

https://kubernetes.io/docs/concepts/configuration/secret/#risks


```
kubectl create secret generic db-secret --from-literal=DB_Host=sql01 --from-literal=DB_User=root --from-literal=DB_Password=password123

```

Reference: 
https://kubernetes.io/docs/concepts/configuration/secret/


#### Multi-containers

```
apiVersion: v1
kind: Pod
metadata:
  name: yellow
spec:
  containers:
    -name
```

Refer k8s - Notes 

#### Init Containers

```
apiVersion: v1
kind: Pod
metadata:
    name: myapp-pod
    labels:
    app: myapp
spec:
    containers:
    - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
    initContainers:
    - name: init-myservice
    image: busybox
    command: ['sh', '-c', 'git clone <some-repository-that-will-be-used-by-application> ; done;']
```

```
apiVersion: v1
kind: Pod
metadata:
    name: myapp-pod
    labels:
    app: myapp
spec:
    containers:
    - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
    initContainers:
    - name: init-myservice
    image: busybox:1.28
    command: ['sh', '-c', 'until nslookup myservice; do echo waiting for myservice; sleep 2; done;']
    - name: init-mydb
    image: busybox:1.28
    command: ['sh', '-c', 'until nslookup mydb; do echo waiting for mydb; sleep 2; done;']
```

busybox image and sleeps for 20 seconds


