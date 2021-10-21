# Docker 帮助文档



帮助文档

https://docs.docker.com/engine/install/

Docker 概述

![image-20210925154100573](C:\Users\HW\AppData\Roaming\Typora\typora-user-images\image-20210925154100573.png)

## 安装Docker

### 0、在ubuntu上安装

**操作系统要求**

Ubuntu Bionic 18.04 (LTS

**环境要求**

**cat /etc/os-release**

系统版本3.3 以上

### 1、卸载旧版本

```
sudo apt-get remove docker docker-engine docker.io containerd runc
```

### 2、安装方法

**使用存储库安装**

#### 设置存储库

在新主机上首次安装 Docker Engine 之前，您需要**设置 Docker 存储库**。之后，您可以从存储库安装和更新 Docker。



```
# 1. 设置存储库

sudo apt-get update

$ sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
    
# 2.添加Docker官方的GPG密钥：

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg


```

#### 安装 Docker 引擎

q安装 Docker-ce

> ```
> 1.更新apt包索引
> 
> sudo apt-get update
>  $ sudo apt-get install docker-ce docker-ce-cli containerd.io
> ```





**从包安装**

**使用便利脚本安装**

### 3、卸载 Docker 引擎

1. 卸载 Docker Engine、CLI 和 Containerd 包：

   ```
   $ sudo apt-get purge docker-ce docker-ce-cli containerd.io
   ```

2. 主机上的映像、容器、卷或自定义配置文件不会自动删除。删除所有镜像、容器和卷：

   ```
   $ sudo rm -rf /var/lib/docker
   $ sudo rm -rf /var/lib/containerd
   ```

