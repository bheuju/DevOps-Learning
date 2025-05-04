### Run on both kubemaster and kubeworker

```bash
echo "$(hostname -I | awk '{print $1}') $(hostname)" | sudo tee -a /etc/hosts
sed -i '/\/swap\.img/s/^/#/' /etc/fstab
echo "net.bridge.bridge-nf-call-iptables  = 1" >> /etc/sysctl.conf
echo "net.bridge.bridge-nf-call-ip6tables = 1" >> /etc/sysctl.conf
echo "net.ipv4.ip_forward                 = 1" >> /etc/sysctl.conf
echo "overlay" > /etc/modules-load.d/k8s.conf
echo "br_netfilter" >> /etc/modules-load.d/k8s.conf

sudo apt-get install -y apt-transport-https ca-certificates curl gnupg
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
sudo chmod 644 /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo chmod 644 /etc/apt/sources.list.d/kubernetes.list

sudo apt update -y
sudo apt -y install docker.io
systemctl enable docker
systemctl start docker
wget https://github.com/Mirantis/cri-dockerd/releases/download/v0.3.17/cri-dockerd_0.3.17.3-0.ubuntu-jammy_amd64.deb
apt-get install -y ./cri-dockerd_0.3.17.3-0.ubuntu-jammy_amd64.deb
sudo apt-get install -y kubectl kubeadm kubelet
systemctl enable kubelet
systemctl start kubelet
```

### Run on both kubemaster

```bash
###ONLY FOR MASTER:
sudo apt-get install -y kubectl kubeadm kubelet

kubeadm init --cri-socket=unix:///var/run/cri-dockerd.sock
curl https://raw.githubusercontent.com/projectcalico/calico/v3.29.3/manifests/calico.yaml -O
kubectl apply -f calico.yaml
```

---

# Kubernetes Dashboard

helm package manager
landscape.cncf.io

## install helm - package manager for kubernetes

https://helm.sh/docs/intro/quickstart/
https://github.com/helm/helm/releases/tag/v3.17.3
amd64 linux release

```
wget https://get.helm.sh/helm-v3.17.3-linux-amd64.tar.gz
tar -xf helm-v3.17.3-linux-amd64.tar.gz
ls
cd linux-amd64
ls
sudo cp helm /usr/bin
helm version
```

https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/

```bash
helm repo list
helm repo add kubernetes-dashboard https://kubernetes.github.io/dashboard/
helm repo list
helm upgrade --install kubernetes-dashboard kubernetes-dashboard/kubernetes-dashboard --create-namespace --namespace kubernetes-dashboard
kubectl -n kubernetes-dashboard port-forward svc/kubernetes-dashboard-kong-proxy 8443:443
# show namespace
kubectl get ns
kubectl get pods -n kubernetes-dashboard
# kubectl verbs resources
# kubectl <action> <resource>
kubectl api-resources
kubectl get all -n kubernetes-dashboard
kubectl get deployment -n kubernetes-dashboard
kubectl get svc -n kubernetes-dashboard

kubectl get svc -A

# -w is watch
kubectl get pods -n kubernetes-dashboard -w
```

```bash
# from pods list
kubectl logs pods/kubernetes-asdasd -n kubernetes-dashboard
```

---

## Run pod

```bash
kubectl run test --image =nginx
kubectl get pods
kubectl logs pods/test
kubectl get pods


mkdir demo
cd demo
# use nginx, dry-run, output to demo.yaml
kubectl run demo --image=nginx --dry-run=server -o yaml > demo.yaml
```

Sample demo.yaml
-> minimum required

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: demo1
  namespace: default
spec:
  containers:
    - image: nginx
      name: demo1
```

### Create Namespace from yaml

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: ntc1

---
apiVersion: v1
kind: Pod
metadata:
  name: demo1
  namespace: default
spec:
  containers:
    - image: nginx
      name: demo1
```

```bash
kubectl create -f demo.yaml
kubectl config get-contexts

# namespace
kubectl get ns
create ns <ns-name>
kubectl get ns

kubectl get pods -n <ns-name>
kubectl config set-context

kubectl delete pod/demo1
```

---

# Lets deploy apps

```bash
kubectl get ns

# Create namespace and set as default namespace
kubectl create ns ntc  # or use yaml that creates namespace

# Set default namespace
kubectl config set-context --current --namespace=ntc
kubectl config get-contexts

kubectl get pods
kubectl logs pods
kubectl get pods -o wide  # shows ip, worker
kubectl describe pod/ntc  # shows details

kubectl run --help
kubectl run
kubectl expose service nignx --port=443 --target-port=8443 --name=nginx-https

# creating service to expose
kubectl create service clusterip nginxsvc --tcp=8443:80 --dry-run=server -o yaml

```

use this `nginxsvc.yaml` to run service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginxsvc
spec:
  selector:
    app.kubernetes.io/name: ntc1
  ports:
    - protocol: TCP
      port: 9376
      targetPort: 80
```

```bash
# pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: demo1
  namespace: default
spec:
  containers:
    - image: nginx
      name: demo1
      ports:
        - containerPort: 80

kubectl create -f pod.yaml
kubectl get pods
kubectl get pods -o wide  # gives ip

curl 172.16.42.19
## welcome to nginx

```

https://spacelift.io/blog/kubernetes-tutorial
This is good blog post describing the terminologies and a bit of theory and some practicals.
