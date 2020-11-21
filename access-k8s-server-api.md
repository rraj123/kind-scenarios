## Access Kubernetes API Server 

### Inspect the kube config file 
<br>

Inspect the kubeconfig file .. 

Here are the key elements
<br>
```
apiVersion:
kind:
clusters:
contexts:
users:
```

<br>

```
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: <data>
    server: https://127.0.0.1:65384
  name: kind-airflow-cluster
contexts:
- context:
    cluster: kind-airflow-cluster
    user: kind-airflow-cluster
  name: kind-airflow-cluster
current-context: kind-airflow-cluster
kind: Config
preferences: {}
users:
- name: kind-airflow-cluster
  user:
    client-certificate-data: <data>
    client-key-data: <data>

```
```
export client=$(grep client-cert ~/.kube/config |cut -d" " -f 6)
 export key=$(grep client-key-data ~/.kube/config |cut -d " " -f 6)
 export auth=$(grep certificate-authority-data ~/.kube/config |cut -d " " -f 6)

echo $client | base64 -d - > ./client.pem
echo $key | base64 -d - > ./client-key.pem
echo $auth | base64 -d - > ./ca.pem

kubectl config view |grep server

curl https://127.0.0.1:65384/version --cert ./client.pem  --key ./client-key.pem --cacert ./ca.pem 

```
To get all the paths 

```
curl https://127.0.0.1:65384/version --cert ./client.pem  --key ./client-key.pem --cacert ./ca.pem 
```
<br>

The output will look like this 


Move this to a file 

[k8 apis list!](k8s-apis.md)   

[link to Google!](http://google.com)

Here are the high-level APIs (first level paths)

```
/api
/apis
/healthz
/livez
/logs
/metrics
/openapi
/readyz
/version
```

What are the core APIs responsible for the K8s systems

```
/api   (core)
/apis  (named)
```

Here is the second level APIs 

```
curl https://127.0.0.1:65384/apis --cert ./client.pem  --key ./client-key.pem --cacert ./ca.pem 

curl https://127.0.0.1:65384/api/v1 --cert ./client.pem  --key ./client-key.pem --cacert ./ca.pem | jq '.resources[] .name'

```
[k8 api/v1 list!](k8s-api-v1.md)  

Core api(s) Link

```
"bindings"
"componentstatuses"
"configmaps"
"endpoints"
"events"
"limitranges"
"namespaces"
"namespaces/finalize"
"namespaces/status"
"nodes"
"nodes/proxy"
"nodes/status"
"persistentvolumeclaims"
"persistentvolumeclaims/status"
"persistentvolumes"
"persistentvolumes/status"
"pods"
"pods/attach"
"pods/binding"
"pods/eviction"
"pods/exec"
"pods/log"
"pods/portforward"
"pods/proxy"
"pods/status"
"podtemplates"
"replicationcontrollers"
"replicationcontrollers/scale"
"replicationcontrollers/status"
"resourcequotas"
"resourcequotas/status"
"secrets"
"serviceaccounts"
"services"
"services/proxy"
"services/status"

```

named Group APIs

```
curl https://127.0.0.1:65384/apis --cert ./client.pem  --key ./client-key.pem --cacert ./ca.pem | jq  '.groups[] .name' 


curl https://127.0.0.1:65384/apis --cert ./client.pem  --key ./client-key.pem --cacert ./ca.pem | jq  '.groups[] .name' | pbcopy
```
<br>
```
"apiregistration.k8s.io"
"extensions"
"apps"
"events.k8s.io"
"authentication.k8s.io"
"authorization.k8s.io"
"autoscaling"
"batch"
"certificates.k8s.io"
"networking.k8s.io"
"policy"
"rbac.authorization.k8s.io"
"storage.k8s.io"
"admissionregistration.k8s.io"
"apiextensions.k8s.io"
"scheduling.k8s.io"
"coordination.k8s.io"
"node.k8s.io"
"discovery.k8s.io"

```

