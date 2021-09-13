# Install Kubernetes Cluster using kubeadm
Follow this documentation to set up a Kubernetes cluster on __CentOS 7__.

This documentation guides you in setting up a cluster with one master node and one worker node.

## Assumptions
|Role|FQDN|IP|OS|RAM|CPU|
|----|----|----|----|----|----|
|Master|kmaster.example.com|172.16.16.100|CentOS 7|2G|2|
|Worker|kworker.example.com|172.16.16.101|CentOS 7|1G|1|

## On both Kmaster and Kworker
Perform all the commands as root user unless otherwise specified
```
sudo su -
yum upgrade -y
```

##### Disable Firewall
```
systemctl disable firewalld; systemctl stop firewalld
```
##### Disable swap
```
swapoff -a; sed -i '/swap/d' /etc/fstab
```
##### Disable SELinux
```
setenforce 0
sed -i --follow-symlinks 's/^SELINUX=enforcing/SELINUX=disabled/' /etc/sysconfig/selinux
```
##### Update sysctl settings for Kubernetes networking
```
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
```
### Kubernetes Setup

Verify the script from here
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

##### Add yum repository
```
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
```
##### Reboot the  system to make the above changes effective, relogin as root 
```
shutdown -r now
```
```
sudo su -
```
##### Install Docker components
```
yum install -y yum-utils
yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
yum install -y docker-ce docker-ce-cli containerd.io
```
##### Install Kubernetes components
```
yum install -y kubelet kubeadm kubectl
```
##### Enable and Start kubelet service
```
systemctl start docker
systemctl enable docker
docker info
systemctl start kubelet
systemctl enable kubelet
exit
sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker
docker run hello-world

cat <<EOF | sudo tee /etc/docker/daemon.json
{
"exec-opts": ["native.cgroupdriver=systemd"]
}
EOF
sudo systemctl daemon-reload
```
##### Reboot the  system to make the above changes effective, relogin as root. Not sure if this step is required, but any way i did it.
```
sudo shutdown -r now
```
```
sudo su -
```
## On kmaster
##### Initialize Kubernetes Cluster
```
kubeadm init --pod-network-cidr=10.244.0.0/16
```
##### To be able to run kubectl commands as non-root user
If you want to be able to run kubectl commands as non-root user, then as a non-root user perform these
```
exit
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

##### Deploy Calico network
https://docs.projectcalico.org/getting-started/kubernetes/self-managed-onprem/onpremises
```
curl https://docs.projectcalico.org/manifests/calico.yaml -O
kubectl apply -f calico.yaml
```
or
##### Deploy Flannel network
https://github.com/flannel-io/flannel#flannel
```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```
##### Cluster join command
```
kubeadm token create --print-join-command
```
## On Kworker
##### Join the cluster
Use the output from __kubeadm token create__ command in previous step from the master server and run here.

## Verifying the cluster
##### Get Nodes status
```
kubectl get nodes
```
##### Get component status
```
kubectl get cs
```
##### If the scheduler is not health do this 
```
sudo vi /etc/kubernetes/manifests/kube-scheduler.yaml
sudo systemctl restart kubelet.service
```
Clear the line (spec->containers->command) containing this phrase: - --port=0

##### Check other stuff
```
kubectl cluster-info
kubectl version --short
```

That's it.

Have Fun!!
