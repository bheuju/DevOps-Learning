# Today's Goal

1. create application `nginx` image
2. push to private repo `i.e., nexus`
3. deploy image to kubernetes

pod
service
replica sets
deployment
storage
load balancer

PV (persistence volume) / PVC

# to debug kube cluster

```
crictl ps -a
```

## Let's start

### Master

```bash
# 1. create namespace and switch it as default namespace
kubectl create ns final
kubectl config set-context --current --namespace=final

# 2. create our folder and create a podfile there
mkdir final
# create podfile
nano pod.yaml
```

`pod.yaml` - `v1`

```yaml
# pod file contents here
# https://kubernetes.io/docs/concepts/workloads/pods/
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
    - name: nginx
      image: nginx:1.14.2
      ports:
        - containerPort: 80
```

```bash
# create application / pod
kubectl create -f pod.yaml

# next we create service
kubectl create service
# but we dont use create service command, we will use yaml file.
# we will add this to s

# https://kubernetes.io/docs/concepts/services-networking/service/
# append to pod.yaml at the end, so the new pod.yaml file is as below
```

`pod.yaml` - `v2`

```yaml
# pod file contents here
# https://kubernetes.io/docs/concepts/workloads/pods/
# this is application part
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app.kubernetes.io/name: nginx
spec:
  containers:
    - name: nginx
      image: nginx:1.14.2
      ports:
        - containerPort: 80

---
# https://kubernetes.io/docs/concepts/services-networking/service/
# this is service part
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
spec:
  selector:
    app.kubernetes.io/name: nginx # this name is name of app from above section
  ports:
    - protocol: TCP
      port: 30007
      targetPort: 80
```

```bash
kubectl create -f pod.yaml

kubectl get pods
# if you want to delete and recreate
kubectl delete -f pod.yaml
kubectl create -f pod.yaml

kubectl get svc
-----------------------------------------------------------------------
NAME        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)     AGE
nginx-svc   ClusterIP   10.96.233.225   <none>        30007/TCP   24s
-----------------------------------------------------------------------

# curl to test if app is running
curl 10.96.233.225:30007  # try curl on cluster ip


# if manually want to label
klubectl label pod nginx app.kubernetes.io/name=nginx1
```

### now we expose to outer world - out of cluster, using nodeport

https://kubernetes.io/docs/concepts/services-networking/service/#nodeport-custom-port

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app.kubernetes.io/name: nginx
spec:
  containers:
    - name: nginx
      image: nginx:1.14.2
      ports:
        - containerPort: 80

---
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
spec:
  type: NodePort # add this
  selector:
    app.kubernetes.io/name: nginx
  ports:
    - protocol: TCP
      nodePort: 30007 # node port change
      port: 80 # add this
      targetPort: 80
```

```bash
# recreate pod
kubectl delete -f pod.yaml
kubectl create -f pod.yaml
```

## now we connect to nexus

https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/#create-a-secret-by-providing-credentials-on-the-command-line

```bash
# add nexus.dev.me to /etc/hosts

# kubectl create secret docker-registry regcred --docker-server=<your-registry-server> --docker-username=<your-name> --docker-password=<your-pword> --docker-email=<your-email>
kubectl create secret docker-registry regcred --docker-server=nexus.dev.me:8082 --docker-username=admin --docker-password=Bishal@123

kubectl get secrets
----------------------------------------------------------
NAME      TYPE                             DATA   AGE
regcred   kubernetes.io/dockerconfigjson   1      58s
----------------------------------------------------------
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mynginx # change name
  labels:
    app.kubernetes.io/name: nginx # change label
spec:
  containers:
    - name: mcon
      image: nexus.dev.me:8082/myniginx:v1.0.0
      ports:
        - containerPort: 80
  imagePullSecrets:
    - name: regcred # use this from kubectl get secrets

---
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
spec:
  type: NodePort # add this
  selector:
    app.kubernetes.io/name: nginx # change label of app
  ports:
    - protocol: TCP
      nodePort: 30007 # node port change
      port: 80 # add this
      targetPort: 80
```

should be accessed from 30007

```bash
# to view details
kubectl describe pod/<podname>

# /etc/containerd/config.toml
registry

```

## Debug nexus pull not working

If image is not getting pulled:
(insecure registries, docker -aG --> in both `master` and `worker`)

1. check nexus.dev.me --> add to `/etc/hosts`
2. check insecure registries
   ```bash
   # cat /etc/docker/daemon.json
   {
    "insecure-registries" : [
        "nexus.dev.me:8082",
        "192.168.22.98:8082"
      ]
   }
   ```
3. try `docker login nexus.dev.me:8082/mynginx:v1.0.0`
4. if success, then try `docker pull nexus.dev.me:8082/mynginx:v1.0.0`
5. also check if `docker ps` works. i.e., `sudo usermod -aG docker <username>`

### Done:

[x] Pods
[x] Service

Next is deployment, replica sets

## Deployment

https://kubernetes.io/docs/concepts/workloads/controllers/deployment/

```bash
cd final
kubectl delete -f pod.yaml


nano deployment.yaml

```

`deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template: # replicas is based on template
    metadata:
      labels:
        app: nginx
        app.kubernetes.io/name: nginx # add this label
    spec:
      containers:
        - name: nginx
          image: nexus.dev.me:8082/mynginx:v1.0.0 # change image name
          ports:
            - containerPort: 80
```

node selector - label worker node. to select to which node to deploy

Add service to this now

`service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
spec:
  type: NodePort
  selector:
    app.kubernetes.io/name: nginx
  ports:
    - protocol: TCP
      nodePort: 30007
      port: 80
      targetPort: 80
```

get all

```bash
kubectl apply -f pod.yaml
kubectl apply -f deployment.yaml
```

---

# For production we use either ingress-controller or load-balancer

We also map domain to the IP

## Ingress Controller

https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/

## Load Balancer

Metal Load Balancer
https://metallb.io/

OpenLB

```bash
kubectl edit configmap -n kube-system kube-proxy

kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.14.9/config/manifests/metallb-native.yaml
```

`lb.yml`

```yaml
# lb.yml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: first-pool
  namespace: metallb-system
spec:
  addresses:
    - 192.168.22.40-192.168.22.50
```

`ll.yml`

```yaml
# ll.yml
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: example
  namespace: metallb-system
spec:
  ipAddressPools:
    - first-pool
```

```bash
kubectl create -f lb.yml
kubectl create -f ll.yml
```

```bash
kubectl get all -n metallb-system
```

`deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
        app.kubernetes.io/name: nginx
    spec:
      containers:
        - name: nginx
          image: nexus.dev.me:8082/mynginx:v1.0.0
          ports:
            - containerPort: 80
      imagePullSecrets:
        - name: repository
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
  annotations:
    metallb.universe.tf/loadBalancerIPs: 192.168.22.42

spec:
  type: LoadBalancer
  selector:
    app.kubernetes.io/name: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```
