# Task 1

### Homework
* Create a deployment nginx. Set up two replicas. Remove one of the pods, see what happens.

###Apply deploy.yaml
```bash
kubectl apply -f deploy.yaml
```

###Get replicas
```bash
kubectl get po
NAME                               READY   STATUS    RESTARTS   AGE
nginx-deployment-8d545c96d-ljfhc   1/1     Running   0          92s
nginx-deployment-8d545c96d-wnppr   1/1     Running   0          92s
```

###Remove one of the pods
```bash
kubectl delete pod nginx-deployment-8d545c96d-ljfhc
pod "nginx-deployment-8d545c96d-ljfhc" deleted
```

###Get replicas
```bash
kubectl get po
NAME                               READY   STATUS    RESTARTS   AGE
nginx-deployment-8d545c96d-t6kk9   1/1     Running   0          30s
nginx-deployment-8d545c96d-wnppr   1/1     Running   0          2m53s
```

We can see that new pod with name "nginx-deployment-8d545c96d-t6kk9" will be created