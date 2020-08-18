# docker学习

## docker三要素

+ 镜像
+ 容器
+ 仓库

## Docker安装

### **安装**

+ 安装命令：`yum -y install docker-io`或`curl -sSL https://get.daocloud.io/docker | sh`

+ **查看版本**
  + 命令：`docker -v`
+ **镜像加速**
  + 在**/etc/docker/**目录下创建daemon.json文件，添加镜像资源，然后重启服务
  + 重新加载文件命令：`systemctl daemon-reload`
  + 重启服务命令：`systemctl restart docker`

```bash
[root@localhost docker]# more daemon.json 
##需要添加的镜像资源
{
    "registry-mirrors": ["https://registry.docker-cn.com"]
}
```

+ **启动docker服务**
  + 命令：`service docker start`

### **查看操作**

+ 命令：`docker ps -a -q`
+ 查看上1次容器命令：`docker ps -l`
+ 查看上3次运行容器：`docker ps -n 3`

### **常用命令**

+ 镜像操作
  + 拉取镜像：`docker pull hello-world`
  + 列出镜像列表：`docker images`
  + 获取最新镜像：`docker pull centos:7.0`
  + 查找镜像：`docker search httpd`
  + 更新镜像：`docker commit -m="提交的信息" -a="作者" e218edb10161 更改后的镜像名`
  + 删除镜像：
    + 删除单个：`docker rmi -f bf756fb1ae65`
    + 删除多个：`docker rmi -f 镜像名1:TAG 镜像名2：TAG`
    + 删除全部：`docker rmi -f $(docker images -qa)`
+ 运行容器
  + 运行容器：`docker run hello-world`
  + 运行容器：` docker run -it REPOSITORY:TAG /bin/bash`
    + -t:在新容器内指定一个伪终端或终端。
    + -i:允许你对容器内的标准输入 (STDIN) 进行交互。
  + 后台启动：`docker run -it -d fantj/ping`
    + 重新启动容器：`docker start 5f1cf7c352d0`
  + 查看正在运行的容器：`docker ps`
  + 停止容器：`docker stop 8d3fdc6c9dcb`
  + 删除容器
    + 删除单个容器：`docker rm 5f1cf7c352d0`
    + 删除多个容器：`docker rm -f $(docker ps -qa)`
+ 查看容器内控制日志
  + 查看日志：`docker logs -f -t --tail id`
    + 例如1：`docker logs 8d3fdc6c9dcb`
    + 例如2：`docker exec -it 4a471223bfc4(为你正在运行容器的id) /bin/bash`
  + 查看docker内地进程：
    + 例子：`docker top 容器id`
+ 退出容器
  + 退出方式1-**容器停止退出**：`exit` 
  + 退出方式2-**容器不停止退出**：`CTRL+P+Q`
+ 进入容器(退出重进)
  + 方式一：`docker attach c4815b08c799 `
  + 方式二（**可在容器外操作容器内**）：`docker exec -t c4815b08c799 /bin/bash`
+ 容器内拷贝文件
  + docker
+ 上传镜像
  + 上传镜像文件：`docker tag docker.io/nginx wnMy/nginx`

+ **运行容器并设置端口**

```bash
#容器内部的 8080 端口映射到我们本地主机的 8000 端口上。
docker run -d -p 8000:8080 tomcat:7

#我们可以通过访问127.0.0.1:8000来访问容器的8080端口
docker run -d -p 127.0.0.1:8000:8080 tomcat:7
```

+ **查看容器端口**
  + 命令：`docker port 634bda3d4e18`
+ **查看正在运行的容器**

```bash
[root@localhost ~]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
8d3fdc6c9dcb        nginx               "/docker-entrypoint.…"   5 seconds ago       Up 4 seconds        80/tcp              practical_ellis
```

+ **删除镜像操作的问题**

```bash
[root@localhost ~]# docker rmi hello-world
Error response from daemon: conflict: unable to remove repository reference "hello-world" (must force) 
- container b3750062c33b is using its referenced image bf756fb1ae65
#该镜像被其他容器所使用，所以需要先删除容器，因此先删除 b3750062c33b，如下命令
[root@localhost ~]# docker rm b3750062c33b
b3750062c33b
#删除所有占用该镜像的容器
[root@localhost ~]# docker rm $(docker ps -a -q)
b65cff8f116a
976df25b9ae3
#可以删除hello-world镜像了
[root@localhost ~]# docker rmi hello-world
Untagged: hello-world:latest
Untagged: hello-world@sha256:49a1c8800c94df04e9658809b006fd8a686cab8028d33cfba2cc049724254202
Deleted: sha256:bf756fb1ae65adf866bd8c456593cd24beb6a0a061dedf42b26a993176745f6b
Deleted: sha256:9c27e219663c25e0f28493790cc0b88bc973ba3b1686355f221c38a36978ac63
#删除方式2
[root@localhost ~]# docker rmi -f hello-world
#删除成功
```

---

### 重要

+ **后台启动**：`docker run -d centos`
+ **重新进入容容器**：`docker attach c4815b08c799`

+ **查看容器内部结构**： `docker inspect c4815b08c799`

+ **生成镜像**：`docker commit -a="wangning" -m="mv webapps" df287de57594 wangning769/mytomcat:1.0`

---

## 安装cent  os

### 启动容器

+ --name：为容器分配别名

+ -i：**以交互模式启动运行容器，通常与`-t`同时使用；**
+ -t：**为容器重新分配一个伪输入中断，通常与`-i`同时使用**
+ -p：**指定启动端口**
+ -P：**随机分配端口**

**启动tomcat**

```bash
#8000为外部访问端口，8080为容器内tomcat端口 访问8000 -> 访问tomcat8080
docker run -it -p 8000:8080 tomcat:7

#随机分配端口
docker run -it -P tomcat:7
```



**启动命令**

```bash
#在linux系统上启动centos容器
[root@localhost ~]# docker run -it centos
#生成新的容器5f1cf7c352d0
#此时，5f1cf7c352d0为docker容器pull下来的centos,而非localhost
[root@5f1cf7c352d0 /]# ps -ef
```

**分配别名启动**

```bash
#分配别名启动 docker run -it --name 别名 服务名
[root@localhost ~]# docker run -it --name mycentos01 centos
[root@d5df29a53042 /]# 
```

**重新启动和关闭**

```bash
#再次启动容器
[root@localhost ~]# docker start mycentos01
mycentos01
[root@localhost ~]# docker ps
CONTAINER ID    IMAGE 	COMMAND		CREATED         STATUS            PORTS               NAMES
c4815b08c799    centos  "/bin/bash" 5 minutes ago   Up 5 minutes                         mycentos01
[root@localhost ~]# docker restart 5f1cf7c352d0
5f1cf7c352d0
[root@localhost ~]# docker ps
CONTAINER ID    IMAGE     COMMAND        CREATED           STATUS          PORTS        NAMES
c4815b08c799    centos   "/bin/bash"     5 minutes ago     Up 5 minutes                 mycentos01
5f1cf7c352d0    centos   "/bin/bash"     27 minutes ago    Up 1 second                 intelligent_shtern

#停止容器
[root@localhost ~]# docker stop 5f1cf7c352d0
5f1cf7c352d0
```

**查看docker运行的容器**

```bash
[root@localhost ~]# docker ps
CONTAINER ID  IMAGE      COMMAND       CREATED        STATUS       PORTS               NAMES
5f1cf7c352d0  centos     "/bin/bash"   2 minutes ago  Up 2 minutes                 intelligent_shtern

```

---

## dockerFile

+ **容器和宿主机数据共享**：`docker run -it -v /myDataVolume:/dataVolumeContainer`

###  保留字指令

> **FROM**：基础镜像，当前新镜像是基于哪个镜像的
>
> **MaAINTAINER**：作者和作者邮箱
>
> **RUN**：容器执行时需要执行的额外命令
>
> **EXPOSE**：对外暴露服务的端口号
>
> **WORKDIR**：指定在创建容器后，中断默认登录的进来的工作目录，一个落脚点
>
> **ENV**：用来在构建时设置环境变量
>
> **ADD**：将宿主机目录下的文件拷贝到容器中，并自动进行解压缩
>
> **COPY**：将文件拷贝到镜像中
>
> **VOLUME**：容器数据卷，用于数据保存和持久化工作
>
> **CMD**：指定一个容器启动时要运行的命令，**多个CMD指令时只有最后一个生效**
>
> **ENTRYPOINT**：指定一个容器启动时要运行的命令
>
> **ONBUILD**：当构建一个被继承的dockerFIle时运行命令，父镜像在被子镜像继承后父镜像的pnbuild被触发

### 自定义镜像

---

+ **编写DockerFile文件**

+ 构建

  + `docker build -t 新镜像名字:TAG .`

+ 运行

  + `docker run -it 新镜像名字:TAG`

  

---

+ **示例1**

```dockerfile
FROM centos

ENV mypath /tmp
WORKDIR $mypath

RUN yum -y install vim
RUN yum -y intall net-tools
EXPOSE 80
CMD /bin/bash
```

+ **生成新镜像**

```bash
docker build -f /mydocker/DockerFile2
```

+ **示例2**

```dockerfile
# volume test
FROM centos
VOLUME ["/dataVolumeContainer1","/dataVolumeContainer2"]
CMD echo "finished,--------success1"
CMD /bin/bash
```

+ **执行结果**

```bash
[root@localhost myDataVolume]# docker build -f /myDataVolume/mydockerfile -t zzyy/centos .
Sending build context to Docker daemon  3.584kB
Step 1/4 : FROM centos
 ---> 831691599b88
Step 2/4 : VOLUME ["/dataVolumeContainer1","/dataVolumeContainer2"]
 ---> Running in bed6d401c40f
Removing intermediate container bed6d401c40f
 ---> 64f6b61b252e
Step 3/4 : CMD echo "finished,--------success1"
 ---> Running in 555b8cc1e385
Removing intermediate container 555b8cc1e385
 ---> 316d97c18600
Step 4/4 : CMD /bin/bash
 ---> Running in 52cc096c04df
Removing intermediate container 52cc096c04df
 ---> ee052c6230d0
Successfully built ee052c6230d0
Successfully tagged zzyy/centos:latest
```

+ **共享路径**

```json
"Mounts": [
            {
                "Type": "volume",
                "Name": "526d4d02b6a64b10bf8d2018d6df201d7f22cbd1f8a7de9c4c3e00a92c46ed03",
                "Source": "/var/lib/docker/volumes/526d4d02b6a64b10bf8d2018d6df201d7f22cbd1f8a7de9c4c3e00a92c46ed03/_data",
                "Destination": "/dataVolumeContainer1",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            },
            {
                "Type": "volume",
                "Name": "05a8912e48a5bef70b10602966ed73bbaaeb016698f6920dbd6ddf429f758c04",
                "Source": "/var/lib/docker/volumes/05a8912e48a5bef70b10602966ed73bbaaeb016698f6920dbd6ddf429f758c04/_data",
                "Destination": "/dataVolumeContainer2",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            }
        ]
```

