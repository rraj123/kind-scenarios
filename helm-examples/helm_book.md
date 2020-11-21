### Chapter 2. Using Helm


```
$ helm repo add bitnami https://charts.bitnami.com/bitnami

$ helm search repo drupal
$ helm search repo content
$ helm search repo drupal --versions
$ helm install mysite bitnami/drupal

```

In Helm 3, naming has been changed. Now instance names are scoped to Kubernetes namespaces. We could install two instances named mysite as long as they each lived in a different namespace.


```
$ kubectl create ns first
$ kubectl create ns second
$ helm install --namespace first mysite bitnami/drupal
$ helm install --namespace second mysite bitnami/drupal
```
This might seem confusing at first, but it becomes clearer when we think about a namespace as a prefix on a name. In that sense, we have a site named “first mysite” and another named “second mysite”.

#### Configuration at Installation Time
You can specify the --values flag multiple times. Some people use this feature to have “common” overrides in one file, and specific overrides in another

There is a second flag that can be used to add individual parameters to an install or upgrade. The --set flag takes one or more values directly. They do not need to be stored in a YAML file.


helm install mysite bitnami/drupal --set drupalUsername=admin


```
helm list
helm list --all-namespaces
helm install mysite bitnami/drupal --set ingress.enabled=false
helm upgrade mysite bitnami/drupal --set ingress.enabled=true

```

#### Listing your installations

```
helm list
helm list --all-namespaces

```
#### Upgrading an Installation

It means upgrading an installation, not a chart.  To modify that installation, use helm upgrade.

+ Upgrade the version of the chart
+ Upgrade the configuration of the installation

```
helm install mysite bitnami/drupal --set ingress.enabled=false

helm upgrade mysite bitnami/drupal --set ingress.enabled=true
```
On occasion, you may want to force one of your services to restart. This is not something you need to use Helm for. You can simply use kubectl itself to restart things. 

```
helm repo update (Fetches the latest package)
helm upgrade mysite bitnami/drupal (update)
```

suggest that you provide consistent configuration with each installation and upgrade. To apply the same configuration to both releases, supply the values on each operation:

```
helm install mysite bitnami/drupal --values values.yaml 
helm upgrade mysite bitnami/drupal --values values.yaml 

```
The above option is the right way to manage the 

This may not be the right way to manage the values..

```
helm upgrade mysite bitnami/drupal --reuse-values
```


#### Uninstalling an installation


```
helm uninstall mysite
```

### Beyond the Basics with Helm

#### Templating and Dry Runs
Helm sends the YAML data to the Kubernetes API server. 

The API server will run a series of checks on the submitted YAML. If Kubernetes accepts the YAML data, Helm will consider the deployment a success. But if Kubernetes rejects the YAML, Helm will exit with an error.

#### Dry-run Flag

```
helm install mysite bitnami/drupal --values values.yaml --set drupalEmail=foo@example.com --dry-run

```

Every release is stored in the secrets 
```
helm upgrade mysite bitnami/drupal
kubectl get secrets 

```

#### To list

```
helm list
helm upgrade wordpress bitnami/wordpress --set image.pullPolicy=NoSuchPolicy
helm ls
```

### HELM get

With helm get, we can closely inspect an individual release. 

#### using HELM get

```
helm get notes mysite
```

#### using HELM values

```
helm get values wordpress
helm get values wordpress --revision 2
```

```
helm get values wordpress --all
```

#### using HELM GET manifests

 helm get manifest. This sub-command retrieves the exact YAML manifest that Helm was produced by the Chart templates.


```
helm get manifest wordpress
# Source: wordpress/charts/mariadb/templates/secrets.yaml
apiVersion: v1
kind: Secret
metadata:
  name: wordpress-mariadb
  labels:
    app: "mariadb"
    chart: "mariadb-7.5.1"
    release: "wordpress"
    heritage: "Helm"
type: Opaque
```
One important detail about this command is that it does not return the current state of all of your resources. It returns the manifest generated from the template. 

If you need to see what is deployed on the cluster, you need to take a look at the 

```
kubectl get secret wordpress-mariadb -o yaml
```

### History and rollbacks

```
helm list
helm history <<release-name>>
helm rollback wordpress 2
helm rollback wordpress <<release-name>>
```

#### Keeping History and Rolling Back
deletion event will destroy all release records associated with that installation. But when --keep-history is specified, you can see the history of an installation even after it has been deleted

```
helm uninstall wordpress --keep-history
helm history wordpress
```


### Deep dive into Installs and upgrades

For example, a Deployment object must have a name unique within its namespace. That is, in the namespace mynamespace I cannot have two Deployments named myapp. But I can have a Deployment and a Pod each named myapp.

```
helm install bitnami/wordpress --generate-name --name-template "foo-{{ randAlpha 9 | lower }}"
```

#### Create a namespace 

```
helm install drupal bitnami/drupal --namespace mynamespace --create-namespace
```

There is not an analogous --delete-namespace on helm uninstall. 

#### Upgrade
Some systems, like CI pipelines, are employed to automatically install or upgrade a chart each time a significant event occurs. 

Careless use of this command could result in overwriting one installation with another. This is why it is not the default behavior for Helm.

```
helm upgrade --install wordpress bitnami/wordpress
helm upgrade --install wordpress bitnami/wordpress
```

#### The --wait and --atomic flags
--wait flag modifies the behavior of the Helm client in a couple of ways.
1. waits for a period of time
2. Sucessful start through by querying kube API server.

--wait can fail for a wide variety of reasons, including network latency, a slow scheduler, busy nodes, slow image pulls, and outright failure of a container to start.

Transient outages

With this in mind, though, helm install --wait is a good tool for making sure that the release is brought all the way to running. But when used in automated systems (like CI), it may cause spurious failures. One recommendation for using --wait in CI is to use a long --timeout (five or ten minutes) to ensure that Kubernetes has time to resolve any transient failures.

A second strategy is to use the --atomic flag instead of the --wait flag. This flag causes the same behavior as --wait unless the release fails. Then, instead of marking the release as failed and exiting, it performs an auto-rollback to the last successful release. In automated systems, the --atomic flag is more resistent to outages, since it is less likely to have a failure as its end result. (Keep in mind, though, that there is no assurance that a rollback will be successful.)

#### Upgrading with --force and --cleanup-on-fail

By design, using --force will cause downtime. While it’s often only seconds of downtime, it is downtime nonetheless. 

It is recommended to only use --force when the situation clearly calls for it, not as a “default option.” For example, the core maintainers do not recommend using --force in CI pipelines that deploy to production.

### Building A Chart

$$  creating a basic fully functional chart


### Building a chart

#### The Chart Creation Command

```
helm create mychar
mychart

├── Chart.yaml 
├── charts 
├── templates 
│   ├── NOTES.txt 
│   ├── _helpers.tpl
│   ├── deployment.yaml
│   ├── ingress.yaml
│   ├── service.yaml
│   ├── serviceaccount.yaml
│   └── tests
│       └── test-connection.yaml 5
└── values.yaml 6

```

```

The Chart.yaml file contains metadata and some functionality controls for the chart.

Dependent charts can optionally be held in the charts directory. Chart dependencies are covered in Chapter 6. For now this will be an empty directory.

Templates used to generate Kubernetes manifests are stored in the templates directory.

The NOTES.txt file is a special template. When a chart is installed the NOTES.txt template is rendered and displayed rather than being installed into a cluster.

Templates can include tests that are not installed as part of the install or upgrade commands. This chart includes a test that is used by the helm test command. Testing is covered in Chapter 7.

Default values passed to the templates when Helm is rendering the manifests are in the values.yaml file. When you instantiate a chart these values can be overridden.
```

#### Values

#### Packaging

#### Linting Charts

### Developing Templates

####

When the curly brackets are used to start and stop actions they can be accompanied by a - to remove leading or trailing white space. The following example illustrates this:

````
{{ "Hello" -}} , {{- "World" }}
````

### Information passes to templates

```
.Release.Name
.Chart.
.Capabilities.
```

```
.Template.Name
```

```
https://github.com/Masterminds/sprig
```

```
https://helm.sh/docs/chart_template_guide/function_list/
```

#### Methods

```
helm template
```

#### Querying Kubernetes Resources In Charts

#### if else 

```
{{- if .Values.ingress.enabled -}}
...
{{- end }}
```

#### Variables

#### Loops

#### dry-run

```
helm install myanvil anvil --dry-run
```


$Chart Testing