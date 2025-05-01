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
