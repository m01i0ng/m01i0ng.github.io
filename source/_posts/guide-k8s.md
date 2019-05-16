---
title: Kubernetes 集群部署踩坑
date: 2019-01-19 21:58:44
tags:
  - K8s
  - Linux
  - Docker
categories: k8s
---

## 环境架构：

### 服务器：

- 192.168.1.220 k8s-master
- 192.168.1.221 k8s-node-1
- 192.168.1.223 k8s-node-2

### 系统：centos 7.4

### 集群部署方式：kubeadm

## 系统配置

### 配置主机映射

所有节点

```bash
cat > /etc/hosts << EOF
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

192.168.1.220 k8s-master
192.168.1.221 k8s-node-1
192.168.1.223 k8s-node-2
EOF
```

### 禁用 selinux

所有节点

```bash
sed -i "s/^SELINUX=enforcing/SELINUX=disabled/g" /etc/sysconfig/selinux
sed -i "s/^SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config
sed -i "s/^SELINUX=permissive/SELINUX=disabled/g" /etc/sysconfig/selinux
sed -i "s/^SELINUX=permissive/SELINUX=disabled/g" /etc/selinux/config
setenforce 0
```

### 禁用防火墙

所有节点

```bash
systemctl disable firewalld && systemctl stop firewalld
```

### 关闭交换分区

所有节点

```bash
sed -i 's/.*swap.*/#&/' /etc/fstab
swapoff -a
```

### 配置转发参数

所有节点

```bash
cat > /etc/sysctl.d/k8s.conf << EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
vm.swappiness=0
EOF
sysctl --system
```

### 加载 ipvs 相关内核模块

所有节点

```bash
# 如果重新开机，需要重新加载
modprobe ip_vs
modprobe ip_vs_rr
modprobe ip_vs_wrr
modprobe ip_vs_sh
modprobe nf_conntrack_ipv4
lsmod | grep ip_vs
```

## 安装配置 Docker

k8s v1.11.0 版本推荐使用 Docker v17.03，经测试 v1.13 也能正常使用，而最新的 v18.05 会产生警告，并无法使用资源限制

### 安装 Docker

所有节点

```bash
yum install -y docker &&\
systemctl enable docker &&\
systemctl start docker
```

### 配置国内镜像

所有节点

```bash
cat > /etc/docker/daemon.json << EOF
{
  "registry-mirrors": ["https://docker.mirrors.ustc.edu.cn/"]
}
EOF
systemctl daemon-reload
systemctl restart docker
```

## 安装配置 Kubernetes

### 安装 Kubernetes

所有节点

```bash
# 配置源
cat > /etc/yum.repos.d/kubernetes.repo << EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

# 安装
yum install -y kubelet kubeadm kubectl ipvsadm
```

### 配置启动 Kubelet

所有节点

```bash
# 配置kubelet使用国内pause镜像
# 配置kubelet的cgroups
# 获取docker的cgroups
DOCKER_CGROUPS=$(docker info | grep 'Cgroup' | cut -d' ' -f3)
echo $DOCKER_CGROUPS
cat > /etc/sysconfig/kubelet << EOF
KUBELET_EXTRA_ARGS="--cgroup-driver=$DOCKER_CGROUPS --pod-infra-container-image=registry.cn-hangzhou.aliyuncs.com/google_containers/pause-amd64:3.1"
EOF

# 启动
systemctl daemon-reload
systemctl enable kubelet && systemctl start kubelet
```

### 配置 master 节点

master 节点

```bash
# 生成配置文件
cat > kubeadm-master.config << EOF
apiVersion: kubeadm.k8s.io/v1alpha2
kind: MasterConfiguration
kubernetesVersion: v1.11.0
imageRepository: registry.cn-hangzhou.aliyuncs.com/google_containers
api:
  # master 节点 ip
  advertiseAddress: 192.168.1.220

controllerManagerExtraArgs:
  node-monitor-grace-period: 10s
  pod-eviction-timeout: 10s

networking:
  podSubnet: 10.244.0.0/16

kubeProxy:
  config:
    # mode: ipvs
    mode: iptables
EOF
```

```bash
# 提前拉取镜像，如果失败可以多次尝试
kubeadm config images pull --config kubeadm-master.config
```

```bash
# 启动
kubeadm init --config kubeadm-master.config
```

如以上命令没有出错，会出现类似 `kubeadm --join xxx` 的说明

```bash
rm -rf $HOME/.kube
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config
```

执行 `kubectl get nodes` 查看节点，此时状态应该为 NotReady，接着开始配置 `flannel` 网络插件

```bash
# 修改镜像
wgte https://raw.githubusercontent.com/coreos/flannel/v0.10.0/Documentation/kube-flannel.yml
sed -i 's/k8s.gcr.io/registry.cn-shanghai.aliyuncs.com\/gcr-k8s/g' kube-flannel.yaml
```

```bash
# 部署
kubectl apply -f kube-flannel.yml
```

查看（需要时间配置，可反复执行第一条查看直到 `pods` 状态全为 `Running`）

```bash
kubectl get po -n kube-system
kubectl get svc -n kube-system
```

配置好网络之后 kubeadm 会自动部署 coredns，再执行 `kubectl get nodes`，此时 `master` 节点状态应该为 `Ready`

### 配置子节点

node-1、node-2 节点

`master` 节点执行 `kubeadm token create --print-join-command`，将结果复制，到 `node` 上执行，加入集群，如果执行失败可能是 `selinux` 没关（我在此处困了很久……），最后在 `master` 节点执行 `kubectl get node`，应该会看见加入进来的节点

## 测试集群

master 节点

部署 deploy

```bash
kubectl run nginx --replicas=2 --image=nginx:alpine --port=80
kubectl expose deployment nginx --type=NodePort --name=test-service-nodeport
kubectl expose deployment nginx --name=test-service

kubectl describe svc test-service
```

访问测试，32340 为上面最后一条命令查看 svc 时获取的端口

浏览器打开 `http://<master-ip>:<32340>` 会返回 nginx 的欢迎界面

清理删除

```bash
kubectl delete svc test-service test-service-nodeport
kubectl delete deploy nginx
```

## 结束

- 部署完成之后想增加节点只要按照子节点部署的步骤进行即可，之后可以参考官方文档安装 `dashboard heapster` 等插件
- 此为单主多子节点配置，多主多子配置方法有所不同，日后再说--
