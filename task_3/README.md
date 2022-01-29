# Task 3

### Homework
* We published minio "outside" using nodePort. Do the same but using ingress.

###Apply configs
```bash
kubectl apply -f pv.yaml
kubectl apply -f pvc.yaml
kubectl apply -f deployment.yaml
kubectl apply -f minio-service.yaml
kubectl apply -f ingress-outside.yaml
```

###Trying to connect ip_minikube
```bash
curl -I -k http://192.168.59.100/
HTTP/1.1 200 OK
Date: Sat, 29 Jan 2022 12:35:19 GMT
Content-Type: text/html
Content-Length: 1356
Connection: keep-alive
Accept-Ranges: bytes
Last-Modified: Sat, 29 Jan 2022 12:35:19 GMT
Vary: Accept-Encoding
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-Xss-Protection: 1; mode=block
```
* Publish minio via ingress so that minio by ip_minikube and nginx returning hostname (previous job) by path ip_minikube/web are available at the same time.
###Delete priveous ingress and apply new config
```bash
kubectl delete -f ingress-outside.yaml
kubectl apply -f ingress.yaml
```

###Trying to connect ip_minikube and we can see redirect ip_minikube/web
```bash
Trying to connect ip_minikube
curl -I -k http://192.168.59.100/
HTTP/1.1 302 Moved Temporarily
Date: Sat, 29 Jan 2022 12:39:07 GMT
Content-Type: text/html
Content-Length: 138
Connection: keep-alive
Location: http://192.168.59.100/web
```

###Trying to connect ip_minikube/web
```bash
curl -I -k http://192.168.59.100/web
HTTP/1.1 200 OK
Date: Sat, 29 Jan 2022 12:39:20 GMT
Content-Type: text/html
Content-Length: 1356
Connection: keep-alive
Accept-Ranges: bytes
Last-Modified: Sat, 29 Jan 2022 12:39:20 GMT
Vary: Accept-Encoding
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-Xss-Protection: 1; mode=block
```

* Create deploy with emptyDir save data to mountPoint emptyDir, delete pods, check data.

###Apply config
```bash
kubectl apply -f deployment-empty_dir.yaml
```

###Create file in volume
```bash
kubectl get po
NAME                     READY   STATUS    RESTARTS   AGE
minio-69f75574bd-mkr26   1/1     Running   0          7s

kubectl exec -it minio-69f75574bd-mkr26 -- bash
touch /data/test_file
```

###Check volume
```bash
ls -la /data/
total 12
drwxrwxrwx 3 root root 4096 Jan 29 13:01 .
drwxr-xr-x 1 root root 4096 Jan 29 13:00 ..
drwxr-xr-x 6 root root 4096 Jan 29 13:00 .minio.sys
-rw-r--r-- 1 root root    0 Jan 29 13:01 test_file
```

###Delete pod
```bash
kubectl delete pod minio-69f75574bd-mkr26
pod "minio-69f75574bd-mkr26" deleted
```

###Get pod name
```bash
kubectl get po
NAME                     READY   STATUS    RESTARTS   AGE
minio-69f75574bd-wmb49   1/1     Running   0          63s
```

###Connect new pod 
```bash
kubectl exec -it minio-69f75574bd-wmb49 -- bash 
```

###See that "test_file" file also be deleted
```bash
ls -la /data/
total 12
drwxrwxrwx 3 root root 4096 Jan 29 13:03 .
drwxr-xr-x 1 root root 4096 Jan 29 13:03 ..
drwxr-xr-x 6 root root 4096 Jan 29 13:03 .minio.sys
```

* Optional. Raise an nfs share on a remote machine. Create a pv using this share, create a pvc for it, create a deployment. Save data to the share, delete the deployment, delete the pv/pvc, check that the data is safe.

###It was optional =)