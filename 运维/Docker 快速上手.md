# Docker 3分钟快速上手

## 一、安装

[https://hub.docker.com/](https://hub.docker.com/)

根据url下载对应系统的docker。 window就下Dockers Desktop

## 二、国内墙

详细请看[https://www.daocloud.io/mirror](https://www.daocloud.io/mirror)

Linux

`curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://f1361db2.m.daocloud.io`

该脚本可以将 --registry-mirror 加入到你的 Docker 配置文件 /etc/docker/daemon.json 中。适用于 Ubuntu14.04、Debian、CentOS6 、CentOS7、Fedora、Arch Linux、openSUSE Leap 42.1，其他版本可能有细微不同。更多详情请访问文档。

### macOS

Docker For Mac

右键点击桌面顶栏的 docker 图标，选择 Preferences ，在 Daemon 标签（Docker 17.03 之前版本为 Advanced 标签）下的 Registry mirrors 列表中加入下面的镜像地址:

`http://f1361db2.m.daocloud.io`

点击 Apply & Restart 按钮使设置生效。

Docker Toolbox 等配置方法请参考[帮助文档](http://guide.daocloud.io/dcs/daocloud-9153151.html#docker-toolbox)。

### Windows

Docker For Windows

在桌面右下角状态栏中右键 docker 图标，修改在 Docker Daemon 标签页中的 json ，把下面的地址:

`http://f1361db2.m.daocloud.io`

加到" `registry-mirrors`"的数组里。点击 Apply 。

Docker Toolbox 等配置方法请参考[帮助文档](http://guide.daocloud.io/dcs/daocloud-9153151.html#docker-toolbox)。

三、安装镜像 

这里演示的是安装elasticsearch

[https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html#docker-cli-run-dev-mode](https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html#docker-cli-run-dev-mode)

### 拉镜像

Obtaining Elasticsearch for Docker is as simple as issuing a `docker pull` command against the Elastic Docker registry.

```
docker pull docker.elastic.co/elasticsearch/elasticsearch:7.0.1
```

Alternatively, you can download other Docker images that contain only features available under the Apache 2.0 license. To download the images, go to [www.docker.elastic.co](https://www.docker.elastic.co/).

运行

Elasticsearch can be quickly started for development or testing use with the following command:

```
docker run -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:7.0.1
```



续：

后面继续放docker镜像的工具 做成快速查找的文档