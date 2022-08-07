# Docker

## 安装

```SHELL
# 卸载旧版本
sudo yum remove docker docker-client docker-client-latest docker-common docker-latest \
                  docker-latest-logrotate docker-logrotate docker-engine
                  
# step 1: 安装必要的一些系统工具
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
# Step 2: 添加软件源信息
sudo yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# Step 3
sudo sed -i 's+download.docker.com+mirrors.aliyun.com/docker-ce+' /etc/yum.repos.d/docker-ce.repo
# Step 4: 更新并安装Docker-CE
sudo yum makecache fast
sudo yum -y install docker-ce
# Step 4: 开启Docker服务
sudo service docker start
sudo systemctl start docker

# 注意：https://developer.aliyun.com/mirror/docker-ce/?spm=a2c6h.25603864.0.0.4ae761d5LAqeto
# 官方软件源默认启用了最新的软件，您可以通过编辑软件源的方式获取各个版本的软件包。例如官方并没有将测试版本的软件源置为可用，您可以通过以下方式开启。同理可以开启各种测试版本等。
# vim /etc/yum.repos.d/docker-ce.repo
#   将[docker-ce-test]下方的enabled=0修改为enabled=1
#
# 安装指定版本的Docker-CE:
# Step 1: 查找Docker-CE的版本:
# yum list docker-ce.x86_64 --showduplicates | sort -r
#   Loading mirror speeds from cached hostfile
#   Loaded plugins: branch, fastestmirror, langpacks
#   docker-ce.x86_64            17.03.1.ce-1.el7.centos            docker-ce-stable
#   docker-ce.x86_64            17.03.1.ce-1.el7.centos            @docker-ce-stable
#   docker-ce.x86_64            17.03.0.ce-1.el7.centos            docker-ce-stable
#   Available Packages
# Step2: 安装指定版本的Docker-CE: (VERSION例如上面的17.03.0.ce.1-1.el7.centos)
# sudo yum -y install docker-ce-[VERSION]

通过运行 hello-world 映像来验证是否正确安装了 Docker Engine-Community 。

$ sudo docker run hello-world

卸载 docker
删除安装包：
yum remove docker-ce
删除镜像、容器、配置文件等内容：
rm -rf /var/lib/docker
rm -rf /var/lib/contained
```



## 配置镜像加速器

可以通过修改daemon配置文件/etc/docker/daemon.json来使用加速器 

```shell
# https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://ur062fot.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker

# 查看加速器配置是否生效
$ docker info
```



## 常用命令

### 帮助启动

```shell
systemctl start docker
systemctl stop docker
systemctl restart docker
#查看状态
systemctl status docker

#开机启动
systemctl enable docker

```

### 镜像命令

```shell
#列出本地主机上的镜像
docker images
-a 列出本地所有镜像
-q 只显示镜像id

#查找镜像
docker search {镜像名}
#只显示前5条结果
docker search --limit 5 {镜像名}

#拉取镜像，没有TAG就是latest最新版
docker pull 镜像名[:TAG]

# 删除镜像  
docker rmi {image_id} 
docker rmi $(docker images -qa)

# 查看 镜像/容器/数据卷 所占的空间
docker system df
```



### 容器

```shell
# 新建+启动容器
docker run [options] IMAGE 
# [options]
--name="容器新名字" 为容器指定一个名称
-d 后台运行容器并返回容器ID
-i 以交互(interactive)模式运行容器。通常与 -t 同时使用
-t 为容器重新分配一个伪输入终端(tty)
-P 随机端口映射
-p 指定端口映射

#
# 放在镜像名后的是 命令，/bin/bash 为交互式shell
# 要退出终端，输入exit
docker run -it ubuntu /bin/bash
# Docker容器后台运行，就必须有一个前台进程；容器运行的如果不是那些一直挂起的命令（如top、tail），就会自动退出
docker run -d redis:6.0.8

# 列出当前运行的所有容器
docker ps
-a 列出当前所有 正在运行的 + 历史上运行过的 容器
-l 显示最近创建的容器
-n 显示最近创建的n个容器
-q 静默模式，只显示容器编号

#退出容器
run进去
exit           退出，容器停止
ctrl + p + q   容器不停止
#重新进入
# 在容器中打开新的终端；用exit退出，会导致容器停止（推荐）
docker exec -it 容器ID /bin/bash
# 直接进入容器启动命令的终端。不会启动新的进程；用exit退出，会导致容器停止
docker attach 容器ID

#一般用-d后台启动程序，再用exec进入对应的容器实例

# 启动已经停止运行的容器
docker start 容器ID或name
# 停止容器
docker stop 容器ID或name
# 强制停止容器
docker kill 容器ID或name

#删除已停止的容器
docker rm [-f] 容器ID
docker rm [-f] $(docker ps -aq)
docker ps -aq | xargs docker rm

#查看容器日志
docker logs 容器ID

# 查看容器内运行的进程
docker top 容器ID
#容器详细信息
docker inspect 容器ID

# 从容器拷贝文件到主机
docker cp 容器id:容器内路径 主机目的路径
#导入/导出容器
# 导出容器的内容留作为一个tar归档文件
docker exoprt 容器ID > xxx.tar
# 从tar包中的内容创建一个新的文件系统，再导入为镜像
cat xxx.tar | docker import - 镜像用户/镜像名:版本号
```

- 

### commit与发布

- 镜像分层

当容器启动时，一个新的可写层被加载到镜像的顶部。这一层通常被称为"容器层"，容器层之下的都叫“镜像层”

所有对容器的改动（添加、删除、修改文件）都只会发生在容器层中。只有容器层是可写的，容器层下面的所有镜像层都是只读的。

Docker中的镜像分层。支持通过扩展现有镜像，创建新的镜像（类似于Java继承一个Base基类，然后自己按需扩展）

```shell

git commit -m="提交描述" -a="作者" 容器ID 包名/要创建的镜像名:版本号

docker commit -m="add vim" -a="xing" 883437a3dba4 atguigu/ubuntu-vim:1.0

# 推送到阿里云 https://cr.console.aliyun.com/repository/
# 容器镜像服务/实例列表/镜像仓库/基本信息
# 将镜像推送到Registry
$ docker login --username=xmanj registry.cn-hangzhou.aliyuncs.com
$ docker tag [ImageId] registry.cn-hangzhou.aliyuncs.com/dokcer-learning/ubuntu-extend:[镜像版本号]
$ docker push registry.cn-hangzhou.aliyuncs.com/dokcer-learning/ubuntu-extend:[镜像版本号]
# docker tag 将原镜像复制为一份符合服务器规范的名字的镜像

#从Registry中拉取镜像
$ docker pull registry.cn-hangzhou.aliyuncs.com/dokcer-learning/ubuntu-extend:[镜像版本号]

```



## 容器数据卷

```shell
# 将docker容器内的数据保存进宿主机的磁盘中

# 特点：
# 1：数据卷可在容器之间共享或重用数据
# 2：卷中的更改可以直接实时生效 (docker cp)
# 3：数据卷中的更改不会包含在镜像的更新中
# 4：数据卷的生命周期一直持续到没有容器使用它为止

$ docker run -it --privileged=true -v /宿主机绝对路径目录:/容器内目录  镜像名
# 使用 --privileged=true 参数，container内的root拥有真正的root权限，否则，container内的root只是外部的一个普通用户权限。

# 查看容器详情，是否挂载成功
$ docker inspect 容器ID

# 读写规则：默认为读写rw，可限制为只读ro
$ docker run -it --privileged=true -v /宿主机绝对路径目录:/容器内目录:[rw/ro]  镜像名
 
 # 容器2继承容器1的卷规则
$ docker run -it  --privileged=true --volumes-from u1 --name u2 ubuntu

```



## DockerFile

Dockerfile是用来构建Docker镜像的构建文件，是由一系列命令和参数构成的脚本。