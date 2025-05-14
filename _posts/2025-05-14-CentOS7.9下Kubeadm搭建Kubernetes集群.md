---
layout: post
title:  "CentOS7.9 下 Kubeadm 搭建 Kubernetes 集群"
date:   2025-05-14 14:00:00
categories: Kubernetes cri-dockerd
tags: Kubernetes Kubeadm cri-dockerd
author: poazy
---

* content
{:toc}
> CentOS7.9 下基于 `kubeadm` 安装 `kubernetes` 集群。







# 环境依赖处理

## 2台服务器(CentOS 7.9)

```bash
192.168.11.167 k8s-master
192.168.11.183 k8s-node1
```

## 设置 `hosts`

- k8s 必须配置（所以机器均作设置）

### 192.168.11.167（Master）

```bash
hostnamectl
hostnamectl set-hostname k8s-master
hostnamectl
```

```bash
# 编辑 hosts 文件添加机器映射
vi /etc/hosts
```

```bash
# 在 hosts 文件中添加以下信息
192.168.11.167 k8s-master
192.168.11.183 k8s-node1
```

### 192.168.11.183（Worker）

```bash
hostnamectl
hostnamectl set-hostname k8s-node1
hostnamectl
```

```bash
# 编辑 hosts 文件添加机器映射
vi /etc/hosts
```

```bash
# 在 hosts 文件中添加以下信息
192.168.11.167 k8s-master
192.168.11.183 k8s-node1
```

## 同步系统时间

- 避免由于 `CentOS 7.9` 系统时间与当前时间的时间差过大引起不必要的错误响应安装

```bash
# 查看时间与时区
timedatectl status
# 设置时区为上海时区
sudo cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

# 同步时间
sudo yum install -y ntp
# 更新时间
sudo ntpdate pool.ntp.org && hwclock -w
```

## CentOS `yum` 更新

```bash
# 更新 yum
sudo yum -y update
```

## 安装依赖

```bash
# 安装依赖包
sudo yum install -y conntrack ipvsadm ipset jq sysstat curl iptables libseccomp wget
```

## 基础配置

- k8s 必须配置（所以机器均作设置）

```bash
# 获取权限，执行下面的命令要权限的
sudo -i

# 1) 关闭防火墙
systemctl stop firewalld && systemctl disable firewalld && systemctl status firewalld

# 2) 关闭selinux
# 将 SELinux 设置为 permissive 模式(将其禁用)
setenforce 0
sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

# 3) 关闭swap
swapoff -a
sed -i '/swap/s/^\(.*\)$/#\1/g' /etc/fstab

# 4) 配置iptables的ACCEPT规则
iptables -F && iptables -X && iptables -F -t nat && iptables -X -t nat && iptables -P FORWARD ACCEPT

# 5) 设置系统参数
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1


EOF
# 应用系统参数
sysctl --system

# 6) 重启
reboot

```



# 安装 Docker

## 安装依懒

```bash
# 安装依赖包
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```

## 配置 Docker `repo`

```bash
# 添加 docker 源
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

## 更新 `yum` 软件源缓存

```bash
sudo yum makecache fast
```

## 安装 Docker

```bash
sudo yum install -y docker-ce docker-ce-cli containerd.io
```

## 启动 Docker

```bash
# 设置 docker 开启启动并启动 docker
sudo systemctl enable docker && systemctl start docker && systemctl status docker
```

## 配置镜像加速

```bash
# 编辑 daemon.json 文件
sudo vi /etc/docker/daemon.json
```

```bash
# 编辑 daemon.json 文件信息添加 registry-mirrors
{
  "exec-opts":["native.cgroupdriver=systemd"],
  "registry-mirrors": [
    "https://mirror.iscas.ac.cn",
    "https://docker.mirrors.ustc.edu.cn",
    "https://docker.nju.edu.cn"
  ]
}
```

```bash
# 保存 daemon.json 文件后
# 重新加载 daemon 配置并重启 docker
sudo systemctl daemon-reload && systemctl restart docker && systemctl status docker
```



# 安装 cri-dockerd

- https://mirantis.github.io/cri-dockerd/usage/install-manually/

## 下载&安装

```bash
wget https://github.com/Mirantis/cri-dockerd/releases/download/v0.3.17/cri-dockerd-0.3.17.amd64.tgz
tar xvf cri-dockerd-0.3.17.amd64.tgz
mv cri-dockerd/cri-dockerd /usr/local/bin/
```

## 创建服务文件

- https://raw.githubusercontent.com/Mirantis/cri-dockerd/master/packaging/systemd/cri-docker.service

```bash
tee /etc/systemd/system/cri-docker.service <<EOF
[Unit]
Description=CRI Interface for Docker Application Container Engine
Documentation=https://docs.mirantis.com
After=network-online.target firewalld.service docker.service
Wants=network-online.target
Requires=cri-docker.socket

[Service]
Type=notify
ExecStart=/usr/local/bin/cri-dockerd --container-runtime-endpoint fd:// --pod-infra-container-image=registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.9
ExecReload=/bin/kill -s HUP $MAINPID
TimeoutSec=0
RestartSec=2
Restart=always

# Note that StartLimit* options were moved from "Service" to "Unit" in systemd 229.
# Both the old, and new location are accepted by systemd 229 and up, so using the old location
# to make them work for either version of systemd.
StartLimitBurst=3

# Note that StartLimitInterval was renamed to StartLimitIntervalSec in systemd 230.
# Both the old, and new name are accepted by systemd 230 and up, so using the old name to make
# this option work for either version of systemd.
StartLimitInterval=60s

# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity

# Comment TasksMax if your systemd version does not support it.
# Only systemd 226 and above support this option.
TasksMax=infinity
Delegate=yes
KillMode=process

[Install]
WantedBy=multi-user.target
EOF
```

## 创建套接字文件

- https://raw.githubusercontent.com/Mirantis/cri-dockerd/master/packaging/systemd/cri-docker.socket

```bash
tee /etc/systemd/system/cri-docker.socket <<EOF
[Unit]
Description=CRI Docker Socket for the API
PartOf=cri-docker.service

[Socket]
ListenStream=%t/cri-dockerd.sock
SocketMode=0660
SocketUser=root
SocketGroup=docker

[Install]
WantedBy=sockets.target
EOF
```

## 启动&开机自动启动

```bash
# 重新加载 daemon 配置并重启 docker
systemctl daemon-reload
# 设置 cri-docker.service 开启启动并启动
systemctl enable cri-docker.service && systemctl start cri-docker.service && systemctl status cri-docker.service
# 设置 cri-docker.socket 开启启动并启动
systemctl enable cri-docker.socket && systemctl start cri-docker.socket && systemctl status cri-docker.socket
```

## 重启并查看状态

```bash
reboot
systemctl status docker
systemctl status cri-docker.service
systemctl status cri-docker.socket
```



# 搭建 K8S 集群

## 安装 kubeXXX 三大组件

### 配置 `yum` 源

- https://mirrors.aliyun.com/kubernetes-new/core/stable/

```bash
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes-new/core/stable/v1.30/rpm/
enabled=1
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes-new/core/stable/v1.30/rpm/repodata/repomd.xml.key
EOF
```

### 安装 `kubeadm`&`kubelet`&`kubectl`

#### 查看版本

```bash
yum list kubeadm --showduplicates | sort -r
```

#### 安装

* `所有服务器`都要安装  kubelet kubeadm kubectl

- 版本选用 `1.30.12`

```bash
yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
```

## 初始化 master 节点

- 192.168.11.167 `仅master服务器执行`

- 可能报错（需要二选一）

```bash
# 如果 初始化命令不小心出错了，可用执行 `kubeadm reset` 命令进行重置后，再次执行初始化命令
[root@k8s-master ~]# kubeadm init
Found multiple CRI endpoints on the host. Please define which one do you wish to use by setting the 'criSocket' field in the kubeadm configuration file: unix:///var/run/containerd/containerd.sock, unix:///var/run/cri-dockerd.sock
To see the stack trace of this error execute with --v=5 or higher
```

```bash
# --cri-socket 参数指定使用 cri-dockerd
kubeadm init --cri-socket=unix:///var/run/cri-dockerd.sock
# 上面命令可能会报超时，添加 --image-repository 参数指定
kubeadm init \
  --kubernetes-version=1.30.12
  --apiserver-advertise-address=192.168.11.167
  --service-cidr=10.96.0.0/12
  --pod-network-cidr=10.244.0.0/16
  --cri-socket=unix:///var/run/cri-dockerd.sock \
  --image-repository registry.cn-hangzhou.aliyuncs.com/google_containers
```

- kubeadm init 成功后，会返回如下信息

```bash
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.11.167:6443 --token 5pwm20.i4n2xv753ha6w6wr \
        --discovery-token-ca-cert-hash sha256:f4f72d50ea2eb190a76c11967871328df592e0c98bdc6061cd49371874714543
```

- 配置 kubectl
```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
export KUBECONFIG=/etc/kubernetes/admin.conf
```

- 设置 kubelet 开启启动并启动

```bash
# 设置 kubelet 开启启动并启动
systemctl enable --now kubelet
# 查看 kubelet 状态
systemctl status kubelet
```

### 安装网络插件

* 安装 `coredns` 所需要的网络插件（只需要在 Master 节点上安装）
* 选择网络插件
  * https://kubernetes.io/docs/concepts/cluster-administration/addons/
* 这里选择 `calico` 作为本次使用的网络插件
  * https://docs.projectcalico.org/v3.25/getting-started/kubernetes/

#### 安装 `calico` 网络插件

##### 先拉取镜像

```bash
sudo docker pull calico/kube-controllers:v3.25.0
sudo docker pull calico/cni:v3.25.0
sudo docker pull calico/node:v3.25.0
```

##### 在 K8S 中安装 `calico`

```bash
# 在 K8S 中安装 calico
kubectl apply -f https://docs.projectcalico.org/v3.25/manifests/calico.yaml
```

##### 检查 POD 的状态

```bash
# 过一会 coredns 的状态正常已经跑起来了
# 确认一下 calico 是否安装成功
kubectl get pods --all-namespaces -w
```

## 加入 worker 节点

- 192.168.11.183 `仅worker服务器执行`

- 添加工作节点可能报错，需要指定 --cri-socket 参数

```bash
[root@k8s-node1 ~]# kubeadm join 192.168.11.167:6443 --token 5pwm20.i4n2xv753ha6w6wr \
>         --discovery-token-ca-cert-hash sha256:f4f72d50ea2eb190a76c11967871328df592e0c98bdc6061cd49371874714543
Found multiple CRI endpoints on the host. Please define which one do you wish to use by setting the 'criSocket' field in the kubeadm configuration file: unix:///var/run/containerd/containerd.sock, unix:///var/run/cri-dockerd.sock
To see the stack trace of this error execute with --v=5 or higher
```

```bash
kubeadm join 192.168.11.167:6443 --token 5pwm20.i4n2xv753ha6w6wr \
        --discovery-token-ca-cert-hash sha256:f4f72d50ea2eb190a76c11967871328df592e0c98bdc6061cd49371874714543 \
        --cri-socket=unix:///var/run/cri-dockerd.sock
```

## 查看集群信息

### 查看集群信息

- 192.168.11.167 `在master服务器执行`

```bash
# 查看下情况，输出 Kubernetes master is running at 192.168.11.167:6443 表示成功
kubectl cluster-info
kubectl get nodes
```

### 查看 POD 情况

- 192.168.11.167 `在master服务器执行`

```bash
# 过一会查看，因为相关组件正在创建启动中
# 查看下 pods 情况，注意 coredns 为 Pending 状态没有启动，需要安装网络插件
kubectl get pods -n kube-system
kubectl get pod -A
kubectl get pods --all-namespaces -w
```

```bash
[root@k8s-master ~]kubectl get pods --all-namespaces -w
NAMESPACE     NAME                                       READY   STATUS    RESTARTS       AGE
kube-system   calico-kube-controllers-5b9b456c66-9xxkz   1/1     Running   0              42m
kube-system   calico-node-kkb7v                          1/1     Running   0              42m
kube-system   calico-node-sq7n8                          1/1     Running   0              42m
kube-system   coredns-6d58d46f65-l29p4                   1/1     Running   0              3h55m
kube-system   coredns-6d58d46f65-ssgxd                   1/1     Running   0              3h55m
kube-system   etcd-k8s-master                            1/1     Running   4 (116m ago)   3h55m
kube-system   kube-apiserver-k8s-master                  1/1     Running   4 (115m ago)   3h55m
kube-system   kube-controller-manager-k8s-master         1/1     Running   4 (116m ago)   3h55m
kube-system   kube-proxy-lr4g2                           1/1     Running   2 (116m ago)   3h50m
kube-system   kube-proxy-zq8s5                           1/1     Running   4 (116m ago)   3h55m
kube-system   kube-scheduler-k8s-master                  1/1     Running   4 (116m ago)   3h55m
```

### 健康检查

- 192.168.11.167 `在master服务器执行`

```bash
curl -k https://localhost:6443/healthz
```

## 安装 Helm

### 下载&安装

```bash
wget https://get.helm.sh/helm-v3.17.3-linux-amd64.tar.gz
tar xvf helm-v3.17.3-linux-amd64.tar.gz
mv linux-amd64/helm /usr/local/bin/
helm version
```

### 添加国内镜像仓库

```bash
helm repo list
helm repo add aliyun https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
helm repo list
# 可以通过 helm repo remove aliyun 命令移除
```

