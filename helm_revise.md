```
helm create mychart2
```

```
helm create mychart
helm create mycharttemp

rm -rf mychart/templates/*



configmap.yaml
~~~~~~~~~~~~~

apiVersion: v1
kind: ConfigMap
metadata:
  name: mychart-configmap
data:
  myvalue: "Sample Config Map"
  
  
  
helm install helm-demo-configmap ./mychart

helm ls

kubectl get all


helm get manifest helm-demo-configmap

kubectl describe configmaps mychart-configmap


helm uninstall helm-demo-configmap
```

### Built in 
There are two ways you can inject
1. values.yaml
2. built-in objects.

```
https://helm.sh/docs/chart_template_guide/builtin_objects/


apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "Sample Config Map"


helm install releasename-test ./mychart

  
helm get manifest releasename-test

helm install --debug --dry-run dryrun-test ./mychart

kubectl describe configmaps releasename-test-configmap

helm uninstall releasename-test
```
### Read value for template


```
mychart/values.yaml
 
 
 costCode: CC98112
 
 
 costCode: {{ .Values.costCode }}
 
 helm install --debug --dry-run firstdryrun ./mychart
 
 
 helm install firstvalue ./mychart
 
 
 helm get manifest firstvalue
 
kubectl describe configmaps firstvalue-configmap
```

### Set values for template

```
 helm install --dry-run --debug --set costCode=CC00000 valueseteg ./mychart
 
 
 
 helm install valueseteg ./mychart --set costCode=CC00000 
 
 helm get manifest valueseteg
 
 kubectl describe configmaps valueseteg-configmap
 
 helm ls
 
helm uninstall valueseteg
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
```

### Template pipeline and default values

```
pipeline
 ~~~~~~~
 
  pipeline: {{ .Values.projectCode | upper | quote }}
  now: {{ now | date "2006-01-02" | quote }}
  

  
  helm install --dry-run --debug valueseteg ./mychart
  
  ~~~~~~~~~~~~~~~~~~~~~~~``
  default value
  
  
  contact: {{ .Values.contact | default "1-800-123-0000" | quote }}
  
 
  
  helm install --dry-run --debug valueseteg ./mychart
  

   
  helm install --dry-run --debug --set contact=1-800-800-8888 valueseteg ./mychart
```

### Flow control 
This is very important step 

Pay attention to the intendation, you may likely to run into an error. 

```
projectCode: aazzxxyy
   infra:
     zone: a,b,c
     region: us-e
   
   
   
 {{ if eq .Values.infra.region "us-e" }}ha: true{{ end }}
 
 
 
 helm install --dry-run --debug controlif ./mychart
 
 
 helm install controlif ./mychart 
 
 
 helm get manifest controlif
 
 helm uninstall controlif 
 
   
   
 //Error
   {{ if eq .Values.infra.region "us-e" }}
     ha: true
   {{ end }}
   
  
    {{ if eq .Values.infra.region "us-e" }}
    ha: true
    {{ end }}
    
    
   
    {{- if eq .Values.infra.region "us-e" }}
    ha: true
    {{- end }}
   
   //Should not be done. Error
   
   {{- if eq .Values.infra.region "us-e" -}}
   ha: true
  {{- end -}}
```

### With block 

```
 tags:
   machine: frontdrive
   rack: 4c
   drive: ssd
   vcard: 8g
   
   
   {{- with .Values.tags }}
   Machine Type: {{ .machine | default "NA" | quote }}
   Rack ID: {{ .rack | quote }}
   Storage Type: {{ .drive | upper | quote }}
   Video Card: {{ .vcard | quote }}
   {{- end }}



~~~~~~~~~~~~~~~~~~~~~~~~



metadata:
  name: {{ .Release.Name}}-configmap
  labels:
  {{- with .Values.tags }}
    first: {{ .machine }}
    second: {{ .rack }}
    third: {{ .drive }}
  {{- end }}


~~~~

  {{- with .Values.tags }}
  Machine Type: {{ .machine | default "NA" | quote }}
  Rack ID: {{ .rack | quote }}
  Storage Type: {{ .drive | upper | quote }}
  Video Card: {{ .vcard | quote }}
  Release: {{ .Release.Name }}
  {{- end }}

# can be solved this issue using variables


~~~~~

  {{- with .Values.tags }}
  Machine Type: {{ .machine | default "NA" | quote }}
  Rack ID: {{ .rack | quote }}
  Storage Type: {{ .drive | upper | quote }}
  Video Card: {{ .vcard | quote }}
  {{- end }}
  Release: {{ .Release.Name }}


helm install --dry-run --debug withscope ./mychart
```



### Range

```
 
  LangUsed:
    - Python
    - Ruby
    - Java
    - Scala
    
  LangUsed: |-
     {{- range .Values.LangUsed }}
     - {{ . | title | quote }}
     {{- end }}   
     
 title - Title case functions
 
 
 helm install --dry-run --debug controlif ./mychart
 ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
 
   NetSpeed: |-
     {{- range tuple "low" "medium" "high" }}
     - {{ . }}
     {{- end }}
  
  
 helm install --dry-run --debug controlif ./mychart
 
 
 ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

```

### Variable

To access values outside of with block, then variables can be used to access the values.

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

#### Names templates

kubernetes templates in templates/ 
except Notes.txt

starting with _may not have manifest

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

Here are the important pointers.. 

1. you can include the template from the same file (Not recommended)
2. create a file name that starts with _<<filename>>.tpl
3. you can include the contents through 
    a. template
    b. include 

template option maintain the intendations
include option maintain does not 

How do you pass the objects to the template '$' or '.'. 


```
 metadata:
   name: {{ .Release.Name}}-configmap
   labels:
 {{ template "mychart.version" . | indent 4 }}
 {{ include "mychart.version" . | indent 4 }}
 data:
   myvalue: "Sample Config Map"
   costCode: {{ .Values.costCode }}
   Zone: {{ quote .Values.infra.zone }}
   Region: {{ quote .Values.infra.region }}
   ProjectCode: {{ upper .Values.projectCode }}
   pipeline: {{ .Values.projectCode | upper | quote }}
   now: {{ now | date "2006-01-02"| quote }}
   contact: {{ .Values.contact | default "1-800-123-0000" | quote }}
   {{- range $key, $value := .Values.tags }}
   {{ $key }}: {{ $value | quote }}
   {{- end }}
{{include "mychart.version" $ | indent 2 }}
```


```
  
   ~~~~~~~~~~~~~~~~~~
   
   _helpers.tpl
   
   
   {{- define "mychart.systemlables" }}
     labels:
       drive: ssd
       machine: frontdrive
       rack: 4c
       vcard: 8g
   {{- end }}
   
   ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
   
   
   app.kubernetes.io/instance: "{{ $.Release.Name }}"
   app.kubernetes.io/version: "{{ $.Chart.AppVersion }}"
   app.kubernetes.io/managed-by: "{{ $.Release.Service }}"
   
   
   {{- template "mychart.systemlables" . }}
   
   {{- template "mychart.systemlables" $ }}
   
helm install --dry-run --debug templatedemo ./mychart
```

#### Notes

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

#### Subcharts

Subchart is similar to charts repository. The subcharts are buried within the charts directory. 


cd to charts

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
 
Here you are dry-running only the subcharts ... not the whole charts.

 helm install --dry-run --debug mysubchart mychart/charts/mysubchart
 
 ~~~~~~~~~~~~~~~~~~~~~
 

 Overriding values from parent
 ~~~~~~~~~~~~~~~~~~~~~~~~~~~
 
 mychart/values.yaml
 
 mysubchart:
   dbhostname: prodmyqlnode
 
overriding from the main driver...

 helm install --dry-run --debug subchartoverride mychart
```


#### Overriding the values..

global is the keyword 

```
 mychart/values.yaml
 
 global:
   orgdomain: com.muthu4all
 
 
 mychart/charts/mysubchart/templates/configmap.yaml
 and
 mychart/templates/configmap.yaml
 
 
 orgdomain: {{ .Values.global.orgdomain }}
 
 helm install --dry-run --debug subchartoverride mychart
 ```

 