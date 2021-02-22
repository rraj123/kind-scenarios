## Kubernetes Topics and approaches

One of the important thing is to understand the kubectl commands and the imperative commands to generate all of the artificats needed for this work.  

The intent is to come up with a scenarios and solve it. 

Pre-requsite:

`kubectl version`

```
$ kubectl version --short

Client Version: v1.19.3
Server Version: v1.19.1
```

Here is the tip for VIM
[VIM Tip](https://opensource.com/article/19/2/getting-started-vim-visual-mode)

Very important steps:

1. Character mode: v (lower-case)
2. Line mode: V (upper-case)
3. Block mode: Ctrl+v

< d + G > will delete from the current line to the end of file

```
# .vimrc
set tabstop=2
set expandtab
set shiftwidth=2
```
##### <i>Restart</i> 
Watch out for `--restartPolicy=`. The default is set to `Always`
For Jobs, you need to have either `restartPolicy:Never` or `restart:OnFailure`




### Core Concept Scenarios

##### Create Pod 

##### Create Pod with Volume

##### Create Pod with Secrets

##### Create Pod with Environment






### Application Lifecycle Scenarios



### Cluster Maintenance Scenarios



### ETCD Scenarios

### Security Scenarios


### Networking Scenarios

1. Ingress Scenarios