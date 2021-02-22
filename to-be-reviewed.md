```
kubectl rolling-update kubia-v1 kubia-v2 --image=luksa/kubia:v2
kubectl rolling-update kubia-v1 kubia-v2 --image=luksa/kubia:v2 --v 6
kubectl create -f kubia-deployment-v1.yaml --record

```

--record command-line option when creating it. This records the command in the revision history, which will be useful later.

kubectl rollout history deployment kubia
 while true; do curl http://130.211.109.222; done
 