# -kubernetes-
ubeadm是官方社区推出的一个用于快速部署kubernetes集群的工具。

这个工具能通过两条指令完成一个kubernetes集群的部署：

    # 创建一个 Master 节点
    $ kubeadm init
    
    # 将一个 Node 节点加入到当前集群中
    $ kubeadm join <Master节点的IP和端口 >

1. 安装要求

在开始之前，部署Kubernetes集群机器需要满足以下几个条件：

- 一台或多台机器，操作系统 CentOS7.x-86_x64
- 硬件配置：2GB或更多RAM，2个CPU或更多CPU，硬盘30GB或更多
- 集群中所有机器之间网络互通
- 可以访问外网，需要拉取镜像
- 禁止swap分区

2. 学习目标

1. 在所有节点上安装Docker和kubeadm
2. 部署Kubernetes Master
3. 部署容器网络插件
4. 部署 Kubernetes Node，将节点加入Kubernetes集群中
5. 部署Dashboard Web页面，可视化查看Kubernetes资源
3. 准备环境

    关闭防火墙：
    $ systemctl stop firewalld
    $ systemctl disable firewalld
    
    关闭selinux：
    $ sed -i 's/enforcing/disabled/' /etc/selinux/config 
    $ setenforce 0
    
    关闭swap：
    $ swapoff -a  $ 临时
    $ vim /etc/fstab  $ 永久
    
    添加主机名与IP对应关系（记得设置主机名）：
    $ cat /etc/hosts
    192.168.31.62 k8s-master
    192.168.31.64 k8s-node1
    192.168.31.66 k8s-node2
    
    将桥接的IPv4流量传递到iptables的链：
    $ cat > /etc/sysctl.d/k8s.conf << EOF
    net.bridge.bridge-nf-call-ip6tables = 1
    net.bridge.bridge-nf-call-iptables = 1
    EOF
    $ sysctl --system

4. 所有节点安装Docker/kubeadm/kubelet

Kubernetes默认CRI（容器运行时）为Docker，因此先安装Docker。

4.1 安装Docker

    $ wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -O /etc/yum.repos.d/docker-ce.repo
    $ yum -y install docker-ce-18.06.1.ce-3.el7
    $ systemctl enable docker && systemctl start docker
    $ docker --version
    Docker version 18.06.1-ce, build e68fc7a

4.2 添加阿里云YUM软件源

    $ cat <<EOF > /etc/yum.repos.d/kubernetes.repo
      [kubernetes]
      name=Kubernetes
      baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
      enabled=1
      gpgcheck=0
      repo_gpgcheck=0
      gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
             http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
      EOF

4.3 安装kubeadm，kubelet和kubectl
由于版本更新频繁，这里指定版本号部署：

    $ yum install -y kubelet-1.13.3 kubeadm-1.13.3 kubectl-1.13.3
    $ systemctl enable kubelet

5. 部署Kubernetes Master

    $ kubeadm init \
      --apiserver-advertise-address=192.168.31.62 \
      --image-repository registry.aliyuncs.com/google_containers \
      --kubernetes-version v1.13.3 \
      --service-cidr=10.1.0.0/16\
      --pod-network-cidr=10.244.0.0/16

由于默认拉取镜像地址k8s.gcr.io国内无法访问，这里指定阿里云镜像仓库地址。

使用kubectl工具：

    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config
    $ kubectl get nodes

6. 安装Pod网络插件（CNI）

    $ kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/a70459be0084506e4ec919aa1c114638878db11b/Documentation/kube-flannel.yml
    
    wget https://raw.githubusercontent.com/coreos/flannel/a70459be0084506e4ec919aa1c114638878db11b/Documentation/kube-flannel.yml
    查看pods镜像
    kubectl get pods -n kube-system 

确保能够访问到quay.io这个registery。

7. 加入Kubernetes Node

向集群添加新节点，执行在kubeadm init输出的kubeadm join命令：

    $  kubeadm join 192.168.229.132:6443 --token cjfvbn.ui5llfjtk8fro3tl --discovery-token-ca-cert-hash sha256:dc31a5eaf14123d7dcb33292bed
    




