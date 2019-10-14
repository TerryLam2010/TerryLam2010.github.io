# Centos 安装 kubectl kubelet kubeadm

安装Docker

#### 卸载旧版本

旧版本的 Docker 称为 `docker` 或者 `docker-engine`，使用以下命令卸载旧版本：

```
$ sudo yum remove docker \
                  docker-common \
                  docker-selinux \
                  docker-engine
```

### 使用 yum 源 安装

执行以下命令安装依赖包：

```
1 sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```

鉴于国内网络问题，强烈建议使用国内源，下面先介绍国内源的使用。

#### 国内源

执行下面的命令添加 `yum` 软件源：

```
$ sudo yum-config-manager \
    --add-repo \
    https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

> 以上命令会添加稳定版本的 Docker CE yum 源。从 Docker 17.06 开始，edge test 版本的 yum 源也会包含稳定版本的 Docker CE。

#### 官方源

```
1 $ sudo yum-config-manager \
2     --add-repo \
3     https://download.docker.com/linux/centos/docker-ce.repo
```

如果需要最新版本的 Docker CE 请使用以下命令：

```
1 $ sudo yum-config-manager --enable docker-ce-edge
2 $ sudo yum-config-manager --enable docker-ce-test
```

#### 安装 Docker CE

更新 `yum` 软件源缓存，并安装 `docker-ce`。

```
1 $ sudo yum makecache fast
2 $ sudo yum install docker-ce
```

### 使用脚本自动安装

在测试或开发环境中 Docker 官方为了简化安装流程，提供了一套便捷的安装脚本，CentOS 系统上可以使用这套脚本安装：

```
1 $ curl -fsSL get.docker.com -o get-docker.sh
2 $ sudo sh get-docker.sh --mirror Aliyun
```

执行这个命令后，脚本就会自动的将一切准备工作做好，并且把 Docker CE 的 edge 版本安装在系统中。



无法访问google ，就拿阿里的

```
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
# 安装
yum install -y kubectl kubelet kubeadm
# 开机启动
systemctl enable kubelet
# 启动
systemctl start kubelet
```







另一个帖子安装单机K8S

# 记一次k8s单机部署

<https://www.jianshu.com/p/d27141e18398> 

## 目录

- [环境]
- [基本安装]
- [k8s集群初始化]
- [安装过程中遇到的问题]
- [安装完成后的配置]
- [配置tomcat RC]
- [配置tomcat service]
- [查看战果]
- [总结]

## 版本记录

| 编号   | 时间       | 备注                               |
| ------ | ---------- | ---------------------------------- |
| v1.0.0 | 2019.02.21 | 文章创建                           |
| v1.0.1 | 2019.03.18 | 目录整改  #基本安装 (补全缺失内容) |

## 开始

> 在开始之前，说一下本次记录的目的 `部署kubernetes单机集群`
> 在理解k8s之前，先不管啥情况，把东西跑起来，与我最熟悉的东西关联起来，这样我个人觉得有助于各种理解，至少比上来接触一堆“奇奇怪怪的概念”要强。

> // todo docker 详细文章（待完成。。。）
> // todo k8s 其他相关内容（待完成。。。）
> // todo 概念还是要懂的，回头在补上 (待完成。。。)

### 环境

基本配置

| 内容     | 参数           |
| -------- | -------------- |
| 操作系统 | centos7        |
| 机器环境 | VMware虚拟机   |
| 部署目标 | tomcat任意版本 |

虚拟机配置

| 内容      | 参数 |
| --------- | ---- |
| 网卡模式  | NAT  |
| CPU核心数 | 2    |
| 内存      | 2G   |

### 基本安装

安装`docker`

```
// 安装docker
$ yum install -y docker-ce
// 开机启动 && 启动服务
$ systemctl enable docker && systemctl start docker
```

安装必要命令

```
# 在/etc/yum.repos.d 下创建k8s.repos, 并添加如下内容

name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
       http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
```

```
// 安装
yum install -y kubelet kubeadm kubectl
systemctl enable kubelet  && systemctl start kubelet
```

拉取镜像

> google 镜像并不在docker库中，因此要防止墙的问题，需要找代替镜像

```
// 查看kubeadm镜像
$ kubeadm config images list

// 结果
k8s.gcr.io/kube-apiserver:v1.13.3
k8s.gcr.io/kube-controller-manager:v1.13.3
k8s.gcr.io/kube-scheduler:v1.13.3
k8s.gcr.io/kube-proxy:v1.13.3
k8s.gcr.io/pause:3.1
k8s.gcr.io/etcd:3.2.24
k8s.gcr.io/coredns:1.2.6

// 执行如下脚本（没有翻墙的同学只能通过阿里云镜像或者其他镜像）
 for i in `kubeadm config images list`; do 
  imageName=${i#k8s.gcr.io/}
  docker pull registry.aliyuncs.com/google_containers/$imageName
  docker tag registry.aliyuncs.com/google_containers/$imageName k8s.gcr.io/$imageName
  docker rmi registry.aliyuncs.com/google_containers/$imageName
done;

// 开机启动 && 启动服务
$ systemctl enable kubelet && systemctl start kubelet
```

### k8s集群初始化

开通必要端口号 防止不必要的问题出现

```
// 6443
firewall-cmd --zone=public --add-port=6443/tcp --permanent && firewall-cmd --reload
// 10250
firewall-cmd --zone=public --add-port=10250/tcp --permanent && firewall-cmd --reload
```

```
// 安装命令
$ kubeadm init
```

### 安装过程中遇到的问题

```
[ERROR NumCPU]: the number of available CPUs 1 is less than the required 2

// 解决：
// 虚拟机修改配置
```

> 
>
> ![img](https://upload-images.jianshu.io/upload_images/15631894-14cd25c81514b3a0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)
>
> image.png

```
[ERROR FileContent--proc-sys-net-bridge-bridge-nf-call-iptables]: /proc/sys/net/bridge/bridge-nf-call-iptables contents are not set to 1

// 解决：
// 编辑配置
$ vi /etc/sysctl.conf
// 添加如下内容
net.bridge.bridge-nf-call-iptables = 1
```

```
[ERROR Swap]: running with swap on is not supported. Please disable swap

// 解决：
// 禁用swap功能
$ swapoff -a

// 修改配置
$ vi /etc/fstab
# 注释如下内容
# k8s need disabled
# /dev/mapper/centos-swap swap                    swap    defaults        0 0
```

安装成功

```
Your Kubernetes master has initialized successfully!
```

### 安装完成后的配置

```
// 安装成功后根据提示配置
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
// master 参与工作（单机模式必备）
kubectl taint nodes --all node-role.kubernetes.io/master-
```

> 这边有一个BUG遇到，实际服务器可能不存在，但是虚拟机使用NAT模式，并且网卡配置为dhcp模式，动态获取ip
> 导致一些里问题
> eg：coredns 启动失败
> eg：kube-apiserver-localhost.localdomain 启动失败
> 解决改为静态配置

```
BOOTPROTO=static

// ...省略...

# ip 为自己实际环境ip
IPADDR=192.168.228.128
GATEWAY=192.168.228.2
NETMASK=255.255.255.0
# DNS酌情处理但是必须要加
DNS1=8.8.8.8
DNS2=114.114.114.114
```

查看k8s集群情况(现在只有system pod)

```
$ kubectl get pods --all-namespaces

// 结果如下
kube-system   coredns-86c58d9df4-48pxx                        1/1     Running   0          6m10s
kube-system   coredns-86c58d9df4-wdlmr                        1/1     Running   0          6m10s
kube-system   etcd-localhost.localdomain                      1/1     Running   0          5m22s
kube-system   kube-apiserver-localhost.localdomain            1/1     Running   0          5m18s
kube-system   kube-controller-manager-localhost.localdomain   1/1     Running   0          5m4s
kube-system   kube-proxy-56m56                                1/1     Running   0          6m10s
kube-system   kube-scheduler-localhost.localdomain            1/1     Running   0          5m18s
kube-system   weave-net-585s5                                 2/2     Running   0          60s
```

开启单机模式

```
$kubectl taint nodes --all node-role.kubernetes.io/master-
```

查看master节点情况

> 不要慌，这里`NotReady` 完全正常
> 安装网卡插件后，查询即可变为`Ready`

```
$ kubectl get nodes
// 结果
localhost.localdomain   NotReady    master   144m   v1.13.3
```

安装网络插件

```
// 这边有很多选择，本次使用`weave`
// 配置地址 https://kubernetes.io/docs/concepts/cluster-administration/addons/

$ kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```

### 配置tomcat RC

配置docker镜像

```
// 查看所需要镜像
docker search tomcat
// 下载tag为tomcat的images（默认版本为lasted）
docker pull tomcat
```

> `replicas: 1` pod实例个数为1
> `image: tomcat` docker镜像
> `name: tomcat-demo` rc名称
> `spec:template:` 当运行实例个数小于replicas时候，rc会根据spec:template: 自动生成对应个数pod

```
apiVersion: v1
kind: ReplicationController
metadata:
  name: tomcat-demo
spec:
  replicas: 1
  selector:
    app: tomcat-demo
  template:
    metadata:
      labels:
        app: tomcat-demo
    spec:
      containers:
      - name: tomcat-demo
        image: tomcat
        ports:
        - containerPort: 8080
```

运行配置并查看结果

```
// 运行yaml
$ kubectl create -f tomcat-demo-rc.yaml
// 结果如下
tomcat-demo   NodePort    10.105.57.5   <none>        8080:30001/TCP   14s
```

### 配置tomcat service

> `nodePort: 30001` 映射端口8080:30001
> `name: tomcat-demo` 服务名

`tomcat-demo-svc.yaml`配置文件内容

```
apiVersion: v1
kind: Service
metadata:
  name: tomcat-demo
spec:
  type: NodePort
  ports:
   - port: 8080
     nodePort: 30001
  selector:
    app: tomcat-demo
```

运行配置并查看结果

```
// 运行yaml
$ kubectl create -f tomcat-demo-svc.yaml
// 结果如下
tomcat-demo   NodePort    10.105.57.5   <none>        8080:30001/TCP   14s
```

> `注意` svc与rc文件可以写在同一个yaml中
> 开通端口号

```
$ firewall-cmd --zone=public --add-port=30001/tcp --permanent && firewall-cmd --reload
```

### 查看战果

浏览器中查看结果 `http://${ip地址}:30001/`

> 
>
> ![img](https://upload-images.jianshu.io/upload_images/15631894-0d97fe380128d28a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)
>
> image.png

### 总结

> 安装过生中遇到不少坑，但是都都克服了，最后加入tomcat环节我觉得很有必要，作为初学者，很多文章，都在讲“k8s集群”， 各种集群部署文章，跟着流程安装了一圈，对错与否都模棱两可，还是需要一个“可视化的结果”