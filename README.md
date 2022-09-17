> # Kubernetes Cluster Using Kubeadm

## How To Create Kubernetes Cluster Using Kubeadm in RedHat Linux Virtual Machines

###  Prerequisites
```
    - 3 or more Virtual Machines ( one master - Two Workers )
    - Kubernets Master Vm Should have min 2 CPU  and 2 GB Ram
```
## The Step 1
### Set Hostname to the VMs
```
    - hostnamectl set-hostname k8smaster (On Master)
    - hostnamectl set-hostname k8sworker1(On Node1)
    - hostnamectl set-hostname k8sworker2 (On Node2)
```
- Applicable For Both Master and Workers Nodes

## The Step 2
### Assign Static IP
```
    - nmcli Command Line 
    - /etc/sysconfig/network-scripts/(interface name)
          - Below is a sample format

            TYPE="Ethernet"
            PROXY_METHOD="none"
            BROWSER_ONLY="no"
            BOOTPROTO="none"
            IPADDR=XXX.XXX.XXX.XXX
            PREFIX=24
            GATEWAY=XXX.XXX.XXX.XXX
            DNS1=192.168.2.254
            DNS2=8.8.8.8
            DNS3=8.8.4.4
            DEFROUTE="yes"
            IPV4_FAILURE_FATAL="no"
            IPV6INIT="no"
            IPV6_AUTOCONF="no"
            IPV6_DEFROUTE="no"
            IPV6_FAILURE_FATAL="no"
            IPV6_ADDR_GEN_MODE="stable-privacy"
            NAME="ens33"
            UUID="Your respective network UUID"
            DEVICE="ens33"
            ONBOOT="yes"

    - Run the command systemctl restart network to restart the network
```
- Applicable For Both Master and Workers Nodess

## The Step 3
### Edit /etc/hosts file
#### Run the below commands on the machines. Change the IP address and host name as per your machine settings. 
```
cat << EOF >> /etc/hosts
192.168.0.xxx k8smaster
192.168.0.xxx k8sworker1
192.168.0.xxx k8sworker2
EOF
```
- Applicable For Both Master and Workers Nodes

## The Step 4
### Disable SELinux
```
setenforce 0
sed -i --follow-symlinks 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
```
- Applicable For Both Master and Workers Nodes

## The Step 5
### Disable firewall and edit Iptables settings
```
systemctl disable firewalld
modprobe br_netfilter
echo '1' > /proc/sys/net/bridge/bridge-nf-call-iptables
```
- Applicable For Both Master and Workers Nodes

## The Step 6
### Setup Kubernetes Repo
```
cat << EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
```
- Applicable For Both Master and Workers Nodes

## The Step 7
### Installing Kubeadm and Docker, Enable and start the services
#### Install Kubeadm
```
yum install kubeadm docker -y
systemctl enable kubelet
systemctl start kubelet
```
#### Install Docker
- Uninstall old versions
```
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```
- Prerequisites
```
sudo yum install -y yum-utils
 sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```
- Install Docker Engine /latest version
```
sudo yum install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```
- Start Docker
```
sudo systemctl enable --now docker.service
```
- Applicable For Both Master and Workers Nodes

## The Step 8
### Disable Swap
```
swapoff -a
vi /etc/fstab and Comment the line with Swap Keyword
```
- Applicable For Both Master and Workers Nodes

## The Step 9
### Initialize Kubernetes Cluster
```
kubeadm init
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
- Applicable For Master Node Only

## The Step 10
### Installing Pod Network using Calico network
```
curl https://docs.projectcalico.org/manifests/calico.yaml -O
kubectl apply -f calico.yaml
kubectl get pods -n kube-system
```
- Applicable For Master Node Only

## The Step 11
### Use the token from Kubeadmin init screen. Below is a sample how it looks like.
```
kubeadm join 192.168.0.xxx:6443 --token XXX
--discovery-token-ca-cert-hash sha256:XX
```

