# Helm / Docker Kubernetes 

```
kubectl config use-context docker-desktop

````
### Install Nginx Ingress Controller


#### Install Config map
```
helm create mychart
helm create mycharttemp

rm -rf mychart/templates/*

helm install <<release-name>> <<your-chart-name>>

configmap.yaml
~~~~~~~~~~~~~

apiVersion: v1
kind: ConfigMap
metadata:
  name: mychart-configmap
data:
  myvalue: "Sample Config Map"
  
helm install <<release-name>> ./mychart
helm ls
kubectl get all
helm get manifest <<release-name>>
kubectl describe configmaps mychart-configmap
````

You can override the values in the values.yaml file using 
--set command
```
helm install valueset ./chart-dir --set costCode=100001
```

### Template functions
```
 chart functions
 ~~~~~~~~~~~~~
 
 https://masterminds.github.io/sprig/
 https://godoc.org/text/template
 
 projectCode: aazzxxyy
 infra:
   zone: a,b,c
   region: us-e
 
 Zone: {{ quote .Values.infra.zone }}
 Region: {{ quote .Values.infra.region }}
 ProjectCode: {{ upper .Values.projectCode }}

helm install --dry-run --debug valueseteg ./mychart
````

### Pipeline functions
There are date functions in the sprig 
 ```
  pipeline: {{ .Values.projectCode | upper | quote }}
  now: {{ now | date "2006-01-02" | quote }}
  
  helm install --dry-run --debug valueseteg ./mychart
  
  default value
  
  contact: {{ .Values.contact | default "1-800-123-0000" | quote }}
  helm install --dry-run --debug valueseteg ./mychart
  helm install --dry-run --debug --set contact=1-800-800-8888 valueseteg ./mychart
  ```

  ### Conditional Statement
   kepping - would eliminate the extra line.. 

  ```
    {{- if eq .Values.infra.region "us-e" }}
    ha: true
    {{- end }}
  ```
### With Scope
 Block level reading 
 within Restricted set of values, the built-in objects cannot be accesed.

```
 tags:
   machine: frontdrive
   rack: 4c
   drive: ssd
   vcard: 8g

In Config
   {{- with .Values.tags }}
   Machine Type: {{ .machine | default "NA" | quote }}
   Rack ID: {{ .rack | quote }}
   Storage Type: {{ .drive | upper | quote }}
   Video Card: {{ .vcard | quote }}
   {{- end }}

helm install --dry-run --debug withscope ./mychart

```

### Range

```

```

### Variable 

```
{{- $relname := .Release.Name -}}
 {{- with .Values.tags }}
 Machine Type: {{ .machine | default "NA" | quote }}
 Rack ID: {{ .rack | quote }}
 Storage Type: {{ .drive | upper | quote }}
 Video Card: {{ .vcard | quote }}
 Release: {{ $relname }}
 {{- end }}
 
 helm install --dry-run --debug vartest ./mychart
 
 ~~~~~~~~~~~~~~
 LangUsed: |-
   {{- range $index, $topping := .Values.LangUsed }}
   - {{ $index }} : {{ $topping }}
   {{- end }} 
 ~~~~~~~~~~~~~~~~~~~~~~~~~
 tags:
   machine: frontdrive
   rack: 4c
   drive: ssd
   vcard: 8g

 tags: 
   {{- range $key, $value := .Values.tags }}
   {{ $key }} : {{ $value }}
   {{- end }} 
 ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
 labels:
   helm.sh/chart: "{{ $.Chart.Name }}-{{ $.Chart.Version }}"
   app.kubernetes.io/instance: "{{ $.Release.Name }}"
   app.kubernetes.io/version: "{{ $.Chart.AppVersion }}"
  app.kubernetes.io/managed-by: "{{ $.Release.Service }}"
```

### Template 

This shows how to templatize it in the same file
```
 {{- define "mychart.systemlables" }}
   labels:
     drive: ssd
     machine: frontdrive
     rack: 4c
     vcard: 8g
 {{- end }}
 
 {{- template "mychart.systemlables" }}
 
 helm install --dry-run --debug templatedemo ./mychart
 ```

### Template - External File

This shows how to externalize the template file. The file name must start with _ (underscore)  eg: _helpers.tpl

To be able to use the built-in object within the template, The template must include the $ (or .)
```
{{- template "mychart.systemlables" $ }}
```
There are two ways of including the templates 
1. template keyword
2. include

 In template keyword option, whatever the template that was in the template option will be taken. The same block could be used in different parts of the yaml, and it may require different indentation. so, you can use other option shown below. 

```
 metadata:
   name: {{ .Release.Name}}-configmap
   labels:
 {{ include "mychart.version" . | indent 4 }}
```

### Notes
```
 templates/NOTES.txt
 
 Thank you for support {{ .Chart.Name }}.
 
 Your release is named {{ .Release.Name }}.
 
 To learn more about the release, try:
 
   $ helm status {{ .Release.Name }}
   $ helm get all {{ .Release.Name }}
   $ helm uninstall {{ .Release.Name }}
 
 ~~~~~~~~~~~~~~~~~
 
helm install notesdemo ./mychart
```

### Subcharts
This must be in the charts directory 
dry run the file 
You can also override the subchart value, 
 + The subchart name must be referred in the yaml. 

```
cd mychart/charts
helm create mysubchart
rm -rf mysubchart/templates/*.*
~~~~~~~~~~~~
mychart/charts/mysubchart/values.yaml
dbhostname: mysqlnode
~~~~~~~~~~
mychart/charts/mysubchart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-innerconfig
data:
  dbhost: {{ .Values.dbhostname }}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~  
helm install --dry-run --debug mysubchart mychart/charts/mysubchart
~~~~~~~~~~~~~~~~~~~~~

Over riding values from parent
~~~~~~~~~~~~~~~~~~~~~~~~~~~
mychart/values.yaml
 
mysubchart:
  dbhostname: prodmyqlnode
helm install --dry-run --debug subchartoverride mychart
```

### Global Subchart.
Common global variable for the chart and the subchart. 
How do you inject the global values across the charts.

This should be part of main value. 

## Repository Management

### Repository hosting options
<li> Http server hosting index.yaml
<li> Multiple Charts with dependency and version can be managed
<li> Can be host of chartmuseum , GCP, S3, github pages

### Install chartmuseum
```
go to chartmuseum 
 curl -LO https://s3.amazonaws.com/chartmuseum/release/latest/bin/linux/amd64/chartmuseum
 
 chmod +x ./chartmuseum
 mv ./chartmuseum /usr/local/bin
 
 chartmuseum --version
 ~~~~~~~~
 chartmuseum --help
 chartmuseum --debug --port=8080 \
   --storage="local" \
  --storage-local-rootdir="./chartstorage"
  
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
helm repo list
helm repo add chartmuseum http://localhost:8080
helm repo list
helm search repo nginx
helm search repo chartmuseum
```

### Add Chart to library
```
 chartmuseum --debug --port=8080 \
   --storage="local" \
  --storage-local-rootdir="./chartstorage"
  
helm repo list
helm repo add mychartmuseumrepo http://192.168.0.52:8080
helm repo list
  
helm search repo mychartmuseumrepo
helm create repotest
helm package repotest/
mkdir mychartdist
mv repotest-0.1.0.tgz mychartdist/
cd mychartdist

curl --data-binary "@repotest-0.1.1.tgz" http://192.168.0.52:8080/api/charts
helm repo list
```

### Create different version of the chart

Here, the version and the app version is can be modified. 

```
 helm package repotest/
 curl --data-binary "@repotest-0.1.1.tgz" http://192.168.0.52:8080/api/charts
 helm search repo mychartmuseumrepo
 helm repo update
 helm search repo mychartmuseumrepo
 This one displays all the versions of the chart. 
 helm search repo -l mychartmuseumrepo
```

### Chart push plug-in

```
helm plugin install https://github.com/chartmuseum/helm-push.git
helm plugin list
helm create helmpushdemo
helm push helmpushdemo/ mychartmuseumrepo
helm repo update
helm search repo mychartmuseumrepo
```

### Chart to github

The repo must contain the index.yaml. The chart musuem creates the index.yaml file automatically, when you push the file. 

before you push to any other repository, you need to create the index.yaml. The below command helps you to create the 


```
helm repo index . 
```

```
mkdir helm_git_repo
cd helm_git_repo
 echo "# helm_repo" >> README.md
 git init
 git add README.md
 git commit -m "first commit"
 git config --global user.email "muthu4all@gmail.com"
 git config --global user.name "Muthukumar"
 git commit -m "first commit"
 git remote add origin https://github.com/<<useraccount>>/helm_git_repo.git
 git push -u origin master
 git remote add origin https://github.com/<<useraccount>>/helm_git_repo.git
git push -u origin master
cd ..
helm create gitrepotest
pwd
cd helm_git_repo
helm package /root/helm_demo/gitrepotest
helm repo index .
git add .
git commit -m "my-repo"
git push -u origin master
https://raw.githubusercontent.com/<<useraccount>>/helm_git_repo/master
```

### Add charts to github repository

```
 helm repo add --username muthu4all@gmail.com --password <<acccess token>> my-github-helm-repo 'https://raw.githubusercontent.com/muthu4all/helm_git_repo/master'
 
 helm search repo my-github-helm-repo
 
 helm repo update
 
helm search repo -l mychartmuseumrepo

helm create gitrepotest2

cd gitrepotest2


cd helm_git_repo
helm package /root/helm_demo/gitrepotest

helm repo index .

git add .

git commit -m "my-repo"

git push -u origin master


 helm repo update
 
helm search repo -l mychartmuseumrepo
```
### Rollback Helm installed chart

```
kubectl get all
helm history install-upgrade-rlbk-demo
helm rollback install-upgrade-rlbk-demo 2
helm history install-upgrade-rlbk-demo
kubectl get all
helm uninstall install-upgrade-rlbk-demo
helm list
kubectl get all
```

### Helm dependency

Keeping too many subcharts are tedious to maintain. So, keeping those charts as dependencies would be a lot easier to maintain.

In the charts.yaml file, add the dependencies blocks 

helm dependency build ./dependencytest

So this way, you can maintain subchart as different project and include it in the main project. 

To update the dependency

helm dependency update ./dependencytest

```
helm create dependencytest
helm install mydeptestinstall --dry-run --debug ./dependencytest
helm install mydeptestinstall ./dependencytest --set service.type=NodePort
 
helm list
helm status mydeptestinstall
kubectl get all


values.yaml

image:
repository: muthu4all/todo
tag: 1.0.0
pullPolicy: IfNotPresent

helm lint ./dependencytest
helm list
helm uninstall mydeptestinstall


helm install mydeptestinstalltodo ./dependencytest --set service.type=NodePort

helm list
kubectl get all
requirements.yaml

dependencies:
  - name: mariadb
    version: 7.x.x
    repository: https://kubernetes-charts.storage.googleapis.com/


helm dependency build ./dependencytest
helm dependency update ./dependencytest
helm uninstall mydeptestinstalltodo
helm install mydepwithmaria ./dependencytest --set service.type=NodePort
kubectl get all
kubectl describe pod <<>>
```

### Chart Hook - Pre and Post Install Hook

Preinstall, Post-install, pre-delete, pre-upgrade, 

```
helm create hooktest
./hooktest/templates
preinstall-hook.yaml

 apiVersion: v1
 kind: Pod
 metadata:
   name: preinstall-hook
   annotations:
     "helm.sh/hook": "pre-install"
 spec:
   containers:
   - name: hook1-container
     image: busybox
     imagePullPolicy: IfNotPresent
     command: ['sh', '-c', 'echo The pre-install hook Pod is running  - preinstall-hook && sleep 10']
   restartPolicy: Never
   terminationGracePeriodSeconds: 0
   
   
 postinstall-hook.yaml  
   
 apiVersion: v1
 kind: Pod
 metadata:
   name: postinstall-hook
   annotations:
     "helm.sh/hook": "post-install"
 spec:
   containers:
   - name: hook2-container
     image: busybox
     imagePullPolicy: IfNotPresent
     command: ['sh', '-c', 'echo post-install hook Pod is running - post-install&& sleep 5']
   restartPolicy: Never
   terminationGracePeriodSeconds: 0  
   
 helm install hooktestinstall ./hooktest  
 helm status hooktestinstall
 kubectl get pods
 kubectl describe pod/preinstall-hook 
 kubectl describe pod/preinstall-hook | grep -E 'Anno|Started:|Finished:'
 
 kubectl describe pod/hooktestinstall-86b894bc76-4jn7c | grep -E 'Anno|Started:|Finished:'
 
 kubectl describe pod/postinstall-hook | grep -E 'Anno|Started:|Finished:'
 
 helm delete hooktestinstall
 kubectl delete pod/preinstall-hook 
kubectl delete pod/postinstall-hook
```

### Kubernetes jobs as Hooks

```
preinstalljob-hook-job.yaml


apiVersion: batch/v1
kind: Job
metadata:
  name: preinstalljob-hook-job
  annotations:
    "helm.sh/hook": "pre-install"

spec:
  template:
    spec:      
      containers:
      - name: pre-install
        image: busybox
        imagePullPolicy: IfNotPresent
        
        command: ['sh', '-c', 'echo pre-install Job Pod is Running ; sleep 3']
    
      restartPolicy: OnFailure
      terminationGracePeriodSeconds: 0
      
  backoffLimit: 3
  completions: 1
  parallelism: 1
  
  
postinstalljob-hook-job.yaml  

apiVersion: batch/v1
kind: Job
metadata:
  name: postinstalljob-hook-job
  annotations:
    "helm.sh/hook": "post-install"
spec:
  template:
    spec:      
      containers:
      - name: post-install
        image: busybox
        imagePullPolicy: IfNotPresent
        
        command: ['sh', '-c', 'echo post-install Pod is Running ; sleep 6']
    
      restartPolicy: OnFailure
      terminationGracePeriodSeconds: 0
      
  backoffLimit: 3
  completions: 1
  parallelism: 1  
  
  
  
  
helm install hooktestjobinstall ./hooktest



kubectl describe pod/preinstalljob-hook-job-m9zlx | grep -E 'Anno|Started:|Finished:'

kubectl describe pod/hooktestjobinstall-5d45b488dc-fjzhk | grep -E 'Anno|Started:|Finished:'

kubectl describe pod/postinstalljob-hook-job-pld5b | grep -E 'Anno|Started:|Finished:'





helm delete hooktestjobinstall

kubectl get jobs  
kubectl delete job/preinstalljob-hook-job
kubectl delete job/postinstalljob-hook-job

```


### Hooks execution as weight

```
reinstalljob-hook-1.yaml

apiVersion: batch/v1
kind: Job
metadata:
  name: preinstalljob-hook-1
  annotations:
    "helm.sh/hook": "pre-install"
    "helm.sh/hook-weight": "-2"

spec:
  template:
    spec:      
      containers:
      - name: pre-install
        image: busybox
        imagePullPolicy: IfNotPresent
        
        command: ['sh', '-c', 'echo pre-install Job Pod is Running Weight -2 and Sleep 2 ; sleep 2']
    
      restartPolicy: OnFailure
      terminationGracePeriodSeconds: 0
      
  backoffLimit: 3
  completions: 1
  parallelism: 1
  
  

preinstalljob-hook-2.yaml

apiVersion: batch/v1
kind: Job
metadata:
  name: preinstalljob-hook-2
  annotations:
    "helm.sh/hook": "pre-install"
    "helm.sh/hook-weight": "3"

spec:
  template:
    spec:      
      containers:
      - name: pre-install
        image: busybox
        imagePullPolicy: IfNotPresent
        
        command: ['sh', '-c', 'echo pre-install Job Pod is Running Weight 3 and Sleep 3 ; sleep 3']
    
      restartPolicy: OnFailure
      terminationGracePeriodSeconds: 0
      
  backoffLimit: 3
  completions: 1
  parallelism: 1



preinstalljob-hook-3.yaml

apiVersion: batch/v1
kind: Job
metadata:
  name: preinstalljob-hook-3
  annotations:
    "helm.sh/hook": "pre-install"
    "helm.sh/hook-weight": "5"

spec:
  template:
    spec:      
      containers:
      - name: pre-install
        image: busybox
        imagePullPolicy: IfNotPresent
        
        command: ['sh', '-c', 'echo pre-install Job Pod is Running Weight 5 and Sleep 5; sleep 5']
    
      restartPolicy: OnFailure
      terminationGracePeriodSeconds: 0
      
  backoffLimit: 3
  completions: 1
  parallelism: 1
  

helm list  
  
helm install hookweightdemo ./hooktest


kubectl describe pod | grep -E 'Name:|Anno|Started:|Finished:'

kubectl describe pod/preinstalljob-hook-1-g78s4 | grep -E 'Anno|Started:|Finished:'
kubectl describe job/preinstalljob-hook-2 | grep -E 'Anno|Started:|Finished:'
kubectl describe job/preinstalljob-hook-3 | grep -E 'Anno|Started:|Finished:'


kubectl describe <<pod name>> | grep -E 'Anno|Started:|Finished:'





helm delete hookweightdemo

kubectl get jobs  
kubectl delete job/preinstalljob-hook-1
kubectl delete job/preinstalljob-hook-2
kubectl delete job/preinstalljob-hook-3

kubectl get all
```


## Helm Testing and verification

### Helm Lint

```
 helm lint upgrade-rlbk/
```

### Helm Hook test

```

```

### Helm Get and status

```

 helm list
 
 helm get notes install-upgrade-rlbk-demo
 
 helm get values install-upgrade-rlbk-demo
 
 helm get manifest install-upgrade-rlbk-demo
 
 helm get hooks install-upgrade-rlbk-demo
 
 
 helm get all install-upgrade-rlbk-demo
 
 
 
helm status install-upgrade-rlbk-demo
```


### Helm Provenance and Integrity

```
 gpg --gen-key
 
 
 
 gpg --list-secret-keys
 
 
 
 
 helm create signchartdemo
 
 ls ~/.gnupg/
 
 helm package --sign --key 'helmkeydemo' --keyring ~/.gnupg/secring.gpg signchartdemo
 
helm verify signchartdemo-0.1.0.tgz
```

## YAML
