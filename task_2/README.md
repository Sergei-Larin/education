# Task 2

## Check env in pod
```bash
kubectl exec -it nginx -- bash
printenv
```

### My output
```bash
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_SERVICE_PORT=443
DATABASE_URL=postgres://connect
HOSTNAME=nginx
PWD=/
PKG_RELEASE=1~bullseye
HOME=/root
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
NJS_VERSION=0.7.2
TERM=xterm
SHLVL=1
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
lastname=lastname
KUBERNETES_SERVICE_HOST=10.96.0.1
KUBERNETES_PORT=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP_PORT=443
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
firstname=firstname
NGINX_VERSION=1.21.6
_=/usr/bin/printenv
root@nginx:/# ent
bash: ent: command not found
root@nginx:/# env
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_SERVICE_PORT=443
DATABASE_URL=postgres://connect
HOSTNAME=nginx
PWD=/
PKG_RELEASE=1~bullseye
HOME=/root
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
NJS_VERSION=0.7.2
TERM=xterm
SHLVL=1
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
lastname=lastname
KUBERNETES_SERVICE_HOST=10.96.0.1
KUBERNETES_PORT=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP_PORT=443
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
firstname=firstname
NGINX_VERSION=1.21.6
_=/usr/bin/env
```

### Create deployment with simple application
```bash
kubectl apply -f nginx-configmap.yaml
kubectl apply -f deployment.yaml
```

### My output
```bash
kubectl get pods -o wide
NAME                   READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES
web-6745ffd5c8-6fgwh   1/1     Running   0          41m   172.17.0.5   minikube   <none>           <none>
web-6745ffd5c8-fds4s   1/1     Running   0          41m   172.17.0.3   minikube   <none>           <none>
web-6745ffd5c8-vt7sh   1/1     Running   0          41m   172.17.0.4   minikube   <none>           <none>
```

* Try connect to pod with curl (curl pod_ip_address). What happens?
* From you PC: not conected
* From minikube (minikube ssh): conected
* From another pod (kubectl exec -it $(kubectl get pod |awk '{print $1}'|grep web-|head -n1) bash): conected


### Create service (ClusterIP)

### My output
```bash
NAME           TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes     ClusterIP   10.96.0.1        <none>        443/TCP        45m
web            ClusterIP   10.111.227.166   <none>        80/TCP         43m
```

* Try connect to service (curl service_ip_address). What happens?

* From you PC: not conencted
* From minikube (minikube ssh) (run the command several times): conected
* From another pod (kubectl exec -it $(kubectl get pod |awk '{print $1}'|grep web-|head -n1) bash) (run the command several times) Connected

### NodePort

### My output
```bash
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP        20h
web          ClusterIP   10.100.170.236   <none>        80/TCP         15m
web-np       NodePort    10.101.147.109   <none>        80:30682/TCP   8s
```

Note how port is specified for a NodePort service
### Checking the availability of the NodePort service type
```bash
minikube ip
curl <minikube_ip>:<nodeport_port>
```

### My output
```bash
minikube ip
192.168.59.100
```

```bash
curl 192.168.59.100:30077
web-6745ffd5c8-fds4s
```

### DNS
Connect to any pod
```bash
cat /etc/resolv.conf
```

### My output
```bash
nameserver 10.96.0.10
search default.svc.cluster.local svc.cluster.local cluster.local
options ndots:5
```

Compare the IP address of the DNS server in the pod and the DNS service of the Kubernetes cluster.
* Compare headless and clusterip
Inside the pod run nslookup to normal clusterip and headless. Compare the results.
You will need to create pod with dnsutils.
```bash
kubectl apply -f https://k8s.io/examples/admin/dns/dnsutils.yaml
```

### [Ingress](https://kubernetes.github.io/ingress-nginx/deploy/#minikube)
Enable Ingress controller

```bash
minikube addons enable ingress
```

### Let's see what the ingress controller creates for us
```bash
kubectl get pods -n ingress-nginx
kubectl get pod $(kubectl get pod -n ingress-nginx|grep ingress-nginx-controller|awk '{print $1}') -n ingress-nginx -o yaml
```

### My output
```bash
NAME                                        READY   STATUS      RESTARTS   AGE
ingress-nginx-admission-create-qx2jr        0/1     Completed   0          51m
ingress-nginx-admission-patch-tbsp6         0/1     Completed   1          51m
ingress-nginx-controller-6d5f55986b-qvnc5   1/1     Running     0          51m
```

### Create Ingress
```bash
kubectl apply -f ingress.yaml
curl $(minikube ip)
```
### My output
```bash
curl 192.168.59.100
web-6745ffd5c8-fds4s
```

### Homework

## In Minikube in namespace kube-system, there are many different pods running. Your task is to figure out who creates them, and who makes sure they are running (restores them after deletion).

```bash
kubectl get all -n kube-system
NAME                                   READY   STATUS    RESTARTS   AGE
pod/coredns-64897985d-k8zqr            1/1     Running   0          68m
pod/etcd-minikube                      1/1     Running   0          68m
pod/kube-apiserver-minikube            1/1     Running   0          68m
pod/kube-controller-manager-minikube   1/1     Running   0          68m
pod/kube-proxy-k8x8d                   1/1     Running   0          68m
pod/kube-scheduler-minikube            1/1     Running   0          68m
pod/storage-provisioner                1/1     Running   0          68m

NAME               TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
service/kube-dns   ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   68m

NAME                        DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
daemonset.apps/kube-proxy   1         1         1       1            1           kubernetes.io/os=linux   68m

NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/coredns   1/1     1            1           68m

NAME                                DESIRED   CURRENT   READY   AGE
replicaset.apps/coredns-64897985d   1         1         1       68m
```

### Answer:

### Controlled by higher level kubernetes like Deployment, Replicaset, Replication controller.

## Implement Canary deployment of an application via Ingress. Traffic to canary deployment should be redirected if you add "canary:always" in the header, otherwise it should go to regular deployment.

### Set to redirect a percentage of traffic to canary deployment.

### Answer:

### Prod deployment
```bash
kubectl apply -f deployment-prod.yaml
kubectl apply -f service-prod.yaml
kubectl apply -f ingress-prod.yaml
```
### Canary deployment
```bash
kubectl apply -f deployment-canary.yaml
kubectl apply -f service-canary.yaml
kubectl apply -f ingress-canary.yaml
```

### See all
```bash
kubectl get all
NAME                              READY   STATUS    RESTARTS   AGE
pod/dnsutils                      1/1     Running   3          22h
pod/web-canary-59dbddbb7f-5f74c   1/1     Running   0          2m40s
pod/web-canary-59dbddbb7f-h6kts   1/1     Running   0          2m40s
pod/web-canary-59dbddbb7f-kxhf6   1/1     Running   0          2m40s
pod/web-prod-bcb5c7b6d-5qf8m      1/1     Running   0          50s
pod/web-prod-bcb5c7b6d-8z2lr      1/1     Running   0          50s
pod/web-prod-bcb5c7b6d-pm5mh      1/1     Running   0          50s

NAME                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP   23h
service/web-canary   ClusterIP   10.107.50.107    <none>        80/TCP    2m40s
service/web-prod     ClusterIP   10.105.223.127   <none>        80/TCP    51s

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/web-canary   3/3     3            3           2m41s
deployment.apps/web-prod     3/3     3            3           51s

NAME                                    DESIRED   CURRENT   READY   AGE
replicaset.apps/web-canary-59dbddbb7f   3         3         3       2m41s
replicaset.apps/web-prod-bcb5c7b6d      3         3         3       51s

kubectl get ingress
NAME           CLASS    HOSTS   ADDRESS     PORTS   AGE
ingress-prod   <none>   *                   80      9s
web-canary     nginx    *       localhost   80      87s
```

### Try connet 10 times to cluster ip via ingress
```bash
for i in $(seq 1 15); do curl -s 192.168.59.100; done
```

### Response mixed traffic:
```bash
web-prod-bcb5c7b6d-8z2lr
web-prod-bcb5c7b6d-5qf8m
web-prod-bcb5c7b6d-pm5mh
web-prod-bcb5c7b6d-pm5mh
web-canary-59dbddbb7f-5f74c
web-prod-bcb5c7b6d-5qf8m
web-prod-bcb5c7b6d-8z2lr
web-prod-bcb5c7b6d-8z2lr
web-prod-bcb5c7b6d-5qf8m
web-prod-bcb5c7b6d-pm5mh
web-canary-59dbddbb7f-h6kts
web-prod-bcb5c7b6d-5qf8m
web-prod-bcb5c7b6d-8z2lr
web-prod-bcb5c7b6d-pm5mh
web-prod-bcb5c7b6d-5qf8m
```

### Add header "canary:always" in request 
```bash
for i in $(seq 1 15); do curl -s -H "canary:always" 192.168.59.100; done
```

### Response only canary traffic:
```bash
web-canary-59dbddbb7f-5f74c
web-canary-59dbddbb7f-h6kts
web-canary-59dbddbb7f-h6kts
web-canary-59dbddbb7f-kxhf6
web-canary-59dbddbb7f-5f74c
web-canary-59dbddbb7f-kxhf6
web-canary-59dbddbb7f-5f74c
web-canary-59dbddbb7f-h6kts
web-canary-59dbddbb7f-kxhf6
web-canary-59dbddbb7f-h6kts
web-canary-59dbddbb7f-5f74c
web-canary-59dbddbb7f-kxhf6
web-canary-59dbddbb7f-h6kts
web-canary-59dbddbb7f-5f74c
web-canary-59dbddbb7f-kxhf6
```