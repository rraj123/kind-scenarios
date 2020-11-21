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

[output]:k8s-apis.md

