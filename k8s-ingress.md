###Ingress Controller

What does ingress controller deployed as ?
What does Ingress does ?

```
kubectl get ingress --all-namespaces
```

Here is the snippet of nginx Ingress controller

```
controlplane $ kubectl -n ingress-space get deployments.apps nginx-ingress-controller -o yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-ingress-controller
  namespace: ingress-space
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      name: nginx-ingress
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        name: nginx-ingress
    spec:
      containers:
      - args:
        - /nginx-ingress-controller
        - --configmap=$(POD_NAMESPACE)/nginx-configuration
        - --default-backend-service=app-space/default-http-backend
        env:
        - name: POD_NAME
          valueFrom:
            fieldRef:
              apiVersion: v1
              fieldPath: metadata.name
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              apiVersion: v1
              fieldPath: metadata.namespace
        image: quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.21.0
        imagePullPolicy: IfNotPresent
        name: nginx-ingress-controller
        ports:
        - containerPort: 80
          name: http
          protocol: TCP
        - containerPort: 443
          name: https
          protocol: TCP
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      serviceAccount: nginx-ingress-serviceaccount
      serviceAccountName: nginx-ingress-serviceaccount
      terminationGracePeriodSeconds: 30
```

Here is the ingress resource 
```
controlplane $ kubectl -n app-space get ing ingress-wear-watch -o yaml
Warning: extensions/v1beta1 Ingress is deprecated in v1.14+, unavailable in v1.22+; use networking.k8s.io/v1 Ingress
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-wear-watch
  namespace: app-space
spec:
  rules:
  - http:
      paths:
      - backend:
          serviceName: wear-service
          servicePort: 8080
        path: /wear
        pathType: ImplementationSpecific
      - backend:
          serviceName: video-service
          servicePort: 8080
        path: /watch
        pathType: ImplementationSpecific
status:
  loadBalancer:
    ingress:
    - {}
controlplane $
```

#### Tip
The ingress resourcs are namespace bound, if you need to create a new mapping, you need to create a new ing resource. 


---------

Deploy Nginx Controller and Ingress. 

```
kubectl create namespace ingress-space

k -n ingress-space create cm nginx-configuration

k -n ingress-space create sa ingress-serviceaccount 

controlplane $ kubectl -n ingress-space get role ingress-role -o yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  creationTimestamp: "2020-12-16T07:34:34Z"
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
  managedFields:
  - apiVersion: rbac.authorization.k8s.io/v1
    fieldsType: FieldsV1
    fieldsV1:
      f:metadata:
        f:labels:
          .: {}
          f:app.kubernetes.io/name: {}
          f:app.kubernetes.io/part-of: {}
      f:rules: {}
    manager: python-requests
    operation: Update
    time: "2020-12-16T07:34:34Z"
  name: ingress-role
  namespace: ingress-space
  resourceVersion: "2026"
  selfLink: /apis/rbac.authorization.k8s.io/v1/namespaces/ingress-space/roles/ingress-role
  uid: a7b415c7-3bf1-4b2a-8c32-8420c74a4200
rules:
- apiGroups:
  - ""
  resources:
  - configmaps
  - pods
  - secrets
  - namespaces
  verbs:
  - get
- apiGroups:
  - ""
  resourceNames:
  - ingress-controller-leader-nginx
  resources:
  - configmaps
  verbs:
  - get
  - update
- apiGroups:
  - ""
  resources:
  - configmaps
  verbs:
  - create
- apiGroups:
  - ""
  resources:
  - endpoints
  verbs:
  - get
controlplane $


-- Create 

```




```
controlplane $ kubectl get rolebindings -n ingress-space ingress-role-binding -o yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  creationTimestamp: "2020-12-16T07:34:34Z"
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
  managedFields:
  - apiVersion: rbac.authorization.k8s.io/v1
    fieldsType: FieldsV1
    fieldsV1:
      f:metadata:
        f:labels:
          .: {}
          f:app.kubernetes.io/name: {}
          f:app.kubernetes.io/part-of: {}
      f:roleRef:
        f:apiGroup: {}
        f:kind: {}
        f:name: {}
      f:subjects: {}
    manager: python-requests
    operation: Update
    time: "2020-12-16T07:34:34Z"
  name: ingress-role-binding
  namespace: ingress-space
  resourceVersion: "2028"
  selfLink: /apis/rbac.authorization.k8s.io/v1/namespaces/ingress-space/rolebindings/ingress-role-binding
  uid: de0734a0-0afc-4ae3-9941-933bc2c43813
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: ingress-role
subjects:
- kind: ServiceAccount
  name: ingress-serviceaccount
controlplane $
```

??? Try to come with a imperative command for service account role binding .. 

```
???
```

Create a deployment (Nginx Controller)

```

$ cat  /root/ingress-controller.yaml
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: ingress-controller
  namespace: ingress-
spec:
  replicas: 1
  selector:
    matchLabels:
      name: nginx-ingress
  template:
    metadata:
      labels:
        name: nginx-ingress
    spec:
      serviceAccountName: ingress-serviceaccount
      containers:
        - name: nginx-ingress-controller
          image: quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.21.0
          args:
            - /nginx-ingress-controller
            - --configmap=$(POD_NAMESPACE)/nginx-configuration
            - --default-backend-service=app-space/default-http-backend
          env:
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
          ports:
            - name: http
                containerPort: 80
            - name: https
              containerPort: 443
```


Expose the service 

```
controlplane $ cat  /var/answers/ingress-service.yaml
---
apiVersion: v1
kind: Service
metadata:
  name: ingress
  namespace: ingress-space
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
    nodePort: 30080
    name: http
  - port: 443
    targetPort: 443
    protocol: TCP
    name: https
  selector:
    name: nginx-ingresscontrolplane $
```

Expose the service 

```
$ cat  /var/answers/ingress-service.yaml
---
apiVersion: v1
kind: Service
metadata:
  name: ingress
  namespace: ingress-space
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
    nodePort: 30080
    name: http
  - port: 443
    targetPort: 443
    protocol: TCP
    name: https
  selector:
    name: nginx-ingress

```


Create the Ingress 
```
$ cat /var/answers/ingress-resource.yaml
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-wear-watch
  namespace: app-space
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
  rules:
  - http:
      paths:
      - path: /wear
        backend:
          serviceName: wear-service
          servicePort: 8080
      - path: /watch
        backend:
          serviceName: video-service
          servicePort: 8080
```