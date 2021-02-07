---
title: 在 Debian 10 和 CentOS 7 中使用阿里云镜像源安装最新版 Docker
date: 2021-01-28 04:10:00
tags:
  - Debian
  - CentOS
categories:
  - 云计算
  - Docker
cover: https://cdn.jsdelivr.net/gh/bianyun1981/CDN@latest/img/posts/2021-01/2021-01-28-033524-671.png
description: Linux发行版自带的 Docker版本一般都比较旧，本文介绍在 Debian 10 和 CentOS 7 中使用阿里云镜像源来安装最新版 Docker，以解决从官方源安装可能会比较慢的问题。
---

{% note warning flat %}
如果之前有安装过旧版本的 Docker (`docker`, `docker.io`, `docker-engine`)，要先卸载后才能安装最新版 Docker。
{% endnote %}

{% note info flat %}
本文撰写时 Debian 的版本为 `10.7`，CentOS 的版本为 `7.9.2009`，Docker的版本为 `20.10.2`。操作系统 或者 Docker 版本不同时，操作过程可能会略有差异。**Debian 和 CentOS 命令不同时，分别在各自的 Tab页分开显示。**

{% endnote %}

{% note info flat %}
出于安全性考虑，一般不用 `root` 用户直接进行操作，而是给普通用户授予 `sudo` 权限，在命令前面加上 `sudo` 来临时提升权限。用 `root` 用户执行以下命令（用实际的用户名替换 `$USER_NAME`）即可给普通用户授予 `sudo` 权限。

{% endnote %}

{% tabs make-user-sudo %}

<!-- tab Debian 10 -->

```bash
# usermod -aG sudo $USER_NAME
```

<!-- endtab -->

<!-- tab CentOS 7 -->

```bash
# usermod -aG wheel $USER_NAME
```

<!-- endtab -->

{% endtabs %}

# 卸载旧版本的 Docker

{% tabs uninstall-old-version %}
<!-- tab Debian 10 -->

```bash
$ sudo apt remove docker \
                  docker-engine \
                  docker.io \
                  containerd \
                  runc
```

<!-- endtab -->

<!-- tab CentOS 7 -->

```bash
$ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

<!-- endtab -->

{% endtabs %}

{% note info flat %}
如果您的系统之前没有安装过旧版本的 Docker，则无需执行上述卸载命令，即便执行了也没有任何副作用，只是会提示所卸载的包不存在。

{% endnote %}

{% note info flat %}
**注意：**上述卸载命令并不会删除路径 `/var/lib/docker/`下的内容，此路径下所包含的 Docker 数据（镜像、容器、卷、网络）可以在新版 Docker下继续使用。最新版 Docker 的安装包名为 `docker-ce`.

{% endnote %}

# 从阿里云镜像源安装 Docker

## 安装必要的系统工具

{% tabs install-essential-utils %}
<!-- tab Debian 10 -->

```bash
$ sudo apt update
$ sudo apt install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
```

<!-- endtab -->

<!-- tab CentOS 7 -->

```bash
$ sudo yum install -y \
    yum-utils \
    device-mapper-persistent-data \
    lvm2
```

<!-- endtab -->

{% endtabs %}

## 安装 GPG 证书

{% tabs install-essential-utils %}
<!-- tab Debian 10 -->

```bash
$ curl -fsSL \
  https://mirrors.aliyun.com/docker-ce/linux/debian/gpg \
  | sudo apt-key add -
```

<!-- endtab -->

<!-- tab CentOS 7 -->

{% note info disabled %}
CentOS 7 不需要安装 GPG 证书
{% endnote %}

<!-- endtab -->

{% endtabs %}

## 添加软件源信息

{% tabs install-essential-utils %}
<!-- tab Debian 10 -->

```bash
$ sudo add-apt-repository \
    "deb [arch=amd64] \
    https://mirrors.aliyun.com/docker-ce/linux/debian \
    $(lsb_release -cs) stable"
```

<!-- endtab -->

<!-- tab CentOS 7 -->

```bash
$ sudo yum-config-manager \
    --add-repo \
    https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

<!-- endtab -->

{% endtabs %}

## 更新并安装 Docker

{% tabs install-essential-utils %}
<!-- tab Debian 10 -->

```bash
$ sudo apt update
$ sudo apt install -y \
    docker-ce \
    docker-ce-cli \
    containerd.io
```

<!-- endtab -->

<!-- tab CentOS 7 -->

```bash
$ sudo yum makecache fast
$ sudo yum install -y \
    docker-ce \
    docker-ce-cli \
    containerd.io
```

<!-- endtab -->

{% endtabs %}

## 验证安装是否成功

{% tabs install-essential-utils %}
<!-- tab Debian 10 -->

```bash
$ sudo docker run --rm hello-world
```

<!-- endtab -->

<!-- tab CentOS 7 -->

```bash
$ sudo systemctl start docker
$ sudo docker run --rm hello-world
```

<!-- endtab -->

{% endtabs %}

如果命令运行后没有报错，且在控制台打印出 hello world 相关的信息，则表明 Docker 已被正确安装。

## 设置普通用户直接运行 docker

非 `root` 用户做如下配置后，就不用在运行 `docker` 命令时在前面加上 `sudo` 了。

```bash
$ sudo group add docker
$ sudo usermod -aG docker $USER
$ newgrp docker
```

然后，用不带 sudo 的 docker 命令验证一下。

```bash
$ docker run --rm hello-world
```

## 配置 Docker 开机自启动

```bash
$ sudo systemctl enable docker.service
$ sudo systemctl enable containerd.service
```

# 配置阿里云镜像加速器

因为网络原因，从 Docker 官方源下载比较大的镜像可能会需要很长的时间，甚至下载失败，好在阿里云容器 Hub 服务提供了官方的镜像站点加速官方镜像的下载速度。使用阿里云的镜像加速器，可以提升获取 Docker 官方镜像的速度。关于加速器的地址，需要先注册阿里云账号，并登录 [容器镜像服务](https://cr.console.aliyun.com/) 的控制台，左侧边栏点击 **镜像中心 => 镜像加速器**，右侧就会显示为你独立分配的加速地址，以及不同系统的配置方法。

```bash
$ sudo mkdir -p /etc/docker
$ sudo tee /etc/docker/daemon.json <<-'EOF'
{
"registry-mirrors": ["https://**xxxxxxxx**.mirror.aliyuncs.com"]
}
EOF
$ sudo systemctl daemon-reload
$ sudo systemctl restart docker
```

到此就算是全部配置完成了，以后再下载 docker 镜像就不会像以前那样慢了。
