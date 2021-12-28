### 简介

众所周知，软件开发中最难搞，也最玄学的就是环境问题了。你可能听过：在我这里运行没问题啊，怎么换个环境就不行了？

所以，针对这一问题，就有了docker。docker可以将项目以及项目所依赖的环境一起打包，然后在各个环境中丝滑地运行。保证了一致性、可移植性以及管理依赖项。可以说，容器不仅可以管理整个项目代码，还可以保存整个项目所依赖的环境，从而得到一个一致且可移植的环境，随时随地重现代码的效果。

docker是一个基于Go语言的应用容器引擎，可以部署到任何设备上，具有很强的便携性。其实容器就是一个正在运行的进程，并且容器进程与容器进程之间是相互独立的，具备很高的隔离性。因为它还应用了一些附加的封装功能，以使其与主机和其他容器保持隔离。

与虚拟机不同，docker是直接运行在宿主机上的，因此容器没有自己的内核。所以与虚拟机相比，docker也更加轻量，没有像虚拟机那样有着厚重的软件、硬件、操作系统。因此，docker非常适合于高密度环境以及中小型部署，而您可以用更少的资源做更多的事情，将服务器的性能可以被压榨到极致。

### 基本概念

Docker中以后三个基本概念:

镜像（Image）：docker 镜像相当于一个类，具备对象所需要的各个属性。镜像用来创建容器, 是容器的只读模板。

容器（Container）：镜像（Image）和容器（Container）的关系，就像是类和实例一样，镜像是静态的定义，而容器是镜像运行时的实体。

仓库（Repository）：仓库可以看做是一个代码存储中心，用来保存镜像。

### centos下docker的安装

卸载旧版本

```shell
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

安装依赖软件包

```shell
sudo yum install -y yum-utils
```

设置镜像仓库，因为默认的国外镜像源可能会比较慢，我们可以设置为国内的，以提高速度。

```shell
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

或者使用国内的阿里云镜像安装

```shell
sudo yum-config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

安装docker CE。因为docker 有两个分支版本：docker CE和docker EE，即社区版和企业版，因为企业版需要官方授权，所以我们一般用社区版。

```shell
sudo yum install docker-ce docker-ce-cli containerd.io
```

启动docker

```shell
sudo systemctl start docker
```

验证是否正确安装

```shell
sudo docker run hello-world
```



### docker操作

#### info

查看docker系统信息，包括镜像和容器数。

```shell
docker info  
```

#### version

查看docker的版本信息。

```shell
docker version  
```

#### run

-d代表以后台的形式启动一个容器，--name则是创建一个容器，并赋予它一个你想要的名字，如果你没有分配，系统会自动给这个容器分配一个名字，并且是唯一的。其中latest可以改为镜像的任意tag，tag可以理解为镜像的不同版本的标签，只要tag存在就可以。如果不加tag，则默认是拉取最新的镜像。另外，docker会先在本地查找是否存在指定镜像，如果有则从本地拉取，没有则从仓库中拉取下来。

```shell
docker run --name mycentos -d centos:latest  
```

-p 是将容器的80端口映射到主机的80端口上，并且对宿主机的目录进行挂载，挂载的格式为： -v  宿主机文件夹路径:容器内文件夹路径。

```shell
docker run -p 80:80 -v /data:/data -d centos:latest 
```

也可以使用交互模式来启动一个容器，一般是通过-it形式来启动。-i是打开交互式窗口，用于与控制台进行交互。-t是分配一个终端，并且在终端退出时，容器仍然会运行。一般在最后要加入一个你想执行的命令，最好是执行完不会退出的，否则容器生成就会闪而过。这里是在容器内执行/bin/bash命令。   

```shell 
docker run -it centos:latest /bin/bash  
```

#### start/stop/restart

命令的功能如其名，分别为开始、停止、重启。

```shell
docker start mycentos
```

```shell
docker stop mycentos
```

```shell
docker restart mycentos
```

#### kill

杀死一个容器，可以以名字杀死，也可以以id杀死。

```shell
docker kill mycentos 
```

如果容器前的若干个字符可以唯一确定容器，那么你就可以只写这几个字符，如下：

```shell
docker kill a39  
```

#### rm

删除容器，可以删除一个容器或者多个容器。

-f 代表强制删除

```shell
docker rm -f db0 fb2 
```

-v 代表删除容器挂载的数据卷。

```shell
docker rm -v mycentos 
```

如果你想删除所有已经停止的容器，那么就运行下面的命令：

```shell
docker rm $(docker ps -a -q)  
```

#### rmi

删除本地一个或多个镜像。参数说明：

- **-f :** 强制删除，如果从该镜像下还有容器正在运行，则无法正常删除，那么就可以使用强制删除；
- **–no-prune :** 不移除该镜像的过程镜像，默认移除；

```shell
 docker rmi -f mycentos 
```

#### docker system prune

最全面的删除：

```shell
docker system prune
```

 这一指令会询问你是否执行_Remove all unused containers, networks, images (both dangling and unreferenced), and optionally,volumes _注意，这个命令会把你暂时关闭的容器，以及暂时没有用到的 Docker 镜像都删掉。

- 添加参数 `--all` 或者 `-a` 来*Remove all unused images not just dangling ones*.
- 添加参数 `--volumes` 来*Remove all volumes not used by at least one container.*

对于volumes的删除：`docker volume prune` Remove all unused local volumes. Unused local volumes are those which are not referenced by any containers.

对于容器的删除：`docker container prune` Remove all stopped containers.

对于镜像的删除：`docker image prune` Remove all dangling images. If `-a` *is specified, will also remove all images not referenced by any container*.

实际上这些指令是一次性删除一类目标，属于范围性操作。而docker提供的那些rm类指令，则更加属于针对性的操作。但是这些针对性操作由于其本身是可以接收多个需要处理的目标的，所以也可以利用迂回的方式进行处理。

- 容器:
  - `docker rm $(docker ps -aq)` : 删除所有停止的容器.
  - `docker container rm` 和 `docker rm` 等价
  - `docker container ls` 和 `docker ps` 等价
- 镜像:
  - `docker rmi $(docker images -f dangling=true -aq)` : 删除所有挂起的镜像
  - `docker image rm`和`docker rmi`等价
  - `docker image ls`和`docker images`等价
- 数据卷Volume:
  - `docker volume rm $(docker volume ls -q)` : 删除不再使用的数据卷。
- build cache:
  - Docker 18.09 引入了 BuildKit ，提升了构建过程的性能、安全、存储管理等能力。
  - `docker builder prune` : 删除 build cache。

#### cp

用于容器与宿主机之间的数据拷贝。参数说明：

- **-L :** 保持源目标中的链接

将主机/www/runoob目录拷贝到容器96f7f14e99ab的/www目录下。

```shell
docker cp /www/runoob 96f7f14e99ab:/www/  
```

将主机/www/runoob目录拷贝到容器96f7f14e99ab中，目录重命名为www。  

```shell
docker cp /www/runoob 96f7f14e99ab:/www  
```

将容器96f7f14e99ab的/www目录拷贝到主机的/tmp目录中。

```shell
docker cp  96f7f14e99ab:/www /tmp/ 
```

#### create

可以使用create命令来创建一个新的容器但不启动它。

```shell
docker create --name mycentos cenots:latest
```

#### exec

如果需要在运行中的容器中执行命令，可以使用exec。执行脚本，执行系统命令都可以。使用方法和run也是大同小异的：**-d :**分离模式: 在后台运行；**-i :** 即使没有附加也保持STDIN 打开；**-t :** 分配一个伪终端。

```shell
docker exec -it mycentos /bin/sh /root/train.sh  
```

```shell
docker exec -it mycentos /bin/bash 
```

```shell
docker exec -it 9af70f8f0714 /bin/bash 
```

#### attach

attach也可以进入正在运行的容器。但是attach退出后会把容器关闭。

```shell
docker attach mycentos
```

#### pause与unpause

可以使用这两个命令来暂停/恢复容器中的所有进程。

```shell
docker pause db0
```

```shell
docker unpause db0  
```

#### images

列出本地镜像。参数说明：

- **-a :** 列出本地所有的镜像（含中间映像层，默认情况下，过滤掉中间映像层）；
- **–digests :** 显示镜像的摘要信息；
- **-f :** 显示满足条件的镜像；
- **–format :** 指定返回值的模板文件；
- **–no-trunc :** 显示完整的镜像信息；
- **-q :** 只显示镜像ID。

查看本地镜像列表。  

```shell
docker images  
```

列出本地镜像中REPOSITORY为centos的镜像列表。  

```shell
docker images  centos  
```

#### ps

查看容器的状态，容器共有七种状态：created（已创建）、restarting（重启中）、running（运行中）、removing（迁移中）、paused（暂停）、exited（停止）、dead（死亡）。

- **-a :** 显示所有的容器，包括未运行的。
- **-f :** 根据条件过滤显示的内容。
- **–format :** 指定返回值的模板文件。
- **-l :** 显示最近创建的容器。
- **-n :** 列出最近创建的n个容器。
- **–no-trunc :** 不截断输出。
- **-q :** 静默模式，只显示容器编号。
- **-s :** 显示总的文件大小。

列出最近创建的10个容器信息。   

```shell
docker ps -n 10
```

 列出所有创建的容器ID。   

```shell
docker ps -a -q  
```

#### inspect

查看容器/镜像的元数据。可选参数：

- **-f :** 指定返回值的模板文件。
- **-s :** 显示总的文件大小。
- **–type :** 为指定类型返回JSON。

```shell
docker inspect  mycentos
```

```shell
docker inspect cenots:latest
```

#### top

查看容器中的进程信息。

```shell
docker top mycentos
```

支持 ps 命令参数。查看所有运行容器的进程信息。  

```shell
for i in  `docker ps |grep Up|awk '{print $1}'`;do echo \ &&docker top $i; done 
```

#### port

列出指定的容器的端口映射。

查看容器mynginx的端口映射情况。 

```shell
docker port mycentos  
```

#### events

获取实时事件。可选参数：

- **-f ：** 根据条件过滤事件；
- **–since ：** 从指定的时间戳后显示所有事件；
- **–until ：** 流水时间显示到指定的时间为止；

显示docker 2016年7月1日后的所有事件。

```shell
docker events  --since="1467302400"  
```

显示docker 镜像为mysql:5.6 2016年7月1日后的相关事件。如果指定的时间是到秒级的，需要将时间转成时间戳。如果时间为日期的话，可以直接使用，如–since=“2016-07-01”。

```shell
docker events -f "image"="mysql:5.6" --since="1467302400" 
```

#### history

查看指定镜像的创建历史。可选参数：

- **-H :** 以可读的格式打印镜像大小和日期，默认为true；
- **–no-trunc :** 显示完整的提交记录；
- **-q :** 仅列出提交记录ID。

```shell
docker history mycentos
```

#### logs

获取容器的日志。可选参数：

- **-f :** 跟踪日志输出
- **–since :** 显示某个开始时间的所有日志
- **-t :** 显示时间戳
- **–tail :** 仅列出最新N条容器日志

跟踪查看容器mycentos的日志输出。 

```shell
docker logs -f mycentos  
```

查看容器mycentos从2021年7月1日后的最新10条日志。

```shell
docker logs --since="2021-07-01" --tail=10 mycentos 
```

#### export

将docker文件系统作为一个tar归档文件导出到STDOUT。可选参数：

- **-o :** 将输入内容写到文件。

将id为a404c6c174a2的容器按日期保存为tar文件。

```shell
docker export -o mysql-`date +%Y%m%d`.tar a404c6c174a2  
```

#### import

从归档的tar文件中创建镜像。可选参数：

- **-c :** 应用docker 指令创建镜像
- **-m :** 提交时的说明文字

从镜像归档文件以创建镜像。

```shell
docker import  mycentos_v3.tar mycentos:v4  
```

#### save

将指定镜像保存成 tar 归档文件。参数说明：

- **-o :** 输出到的文件。

将镜像centos生成mycentos.tar文档 

```shell
docker save -o mycentos.tar centos  
```

#### load

导入使用 **docker save** 命令导出的镜像。参数说明：

- **–input , -i :** 指定导入的文件，代替 STDIN。
- **–quiet , -q :** 精简输出信息。

导入镜像  

```shell
docker load --input mycentos.tar  
```

#### commit

从容器创建一个新的镜像，也就是将容器保存为新的镜像，一般指定容器ID就可以提交了。参数说明：

- **-a :** 提交的镜像作者；
- **-c :** 使用Dockerfile指令来创建镜像；
- **-m :** 提交时的说明文字；
- **-p :** 在commit时，将容器暂停。

将容器a404c6c174a2 保存为新的镜像,并添加提交人信息和说明信息。 

```shell
docker commit -a "guodong" -m "my db" a404c6c174a2  mymysql:v1   
```

#### tag

标记本地镜像，将其归入某一仓库。

将镜像ubuntu:15.10标记为 runoob/ubuntu:v3 镜像。  

```shell
docker tag ubuntu:15.10 runoob/ubuntu:v3  
```

#### login/logout

**docker login :** 登陆到一个Docker镜像仓库，如果未指定镜像仓库地址，默认为官方仓库 Docker Hub

**docker logout :** 登出一个Docker镜像仓库，如果未指定镜像仓库地址，默认为官方仓库 Docker Hub参数说明：

- **-u :** 登陆的用户名
- **-p :** 登陆的密码

登陆到docker Hub 

```shell
docker login -u 用户名 -p 密码  
```

登出docker Hub 

```shell
docker logout 
```

#### pull

从镜像仓库中拉取或者更新指定镜像。参数说明：

- **-a :** 拉取所有 tagged 镜像
- **–disable-content-trust :** 忽略镜像的校验,默认开启

从docker Hub下载centos最新版镜像。

```shell
docker pull centos  
```

从docker Hub下载REPOSITORY为centos的所有镜像。 

```shell
docker pull -a centos  
```

#### push

将本地的镜像上传到镜像仓库,要先登陆到镜像仓库。参数说明：

- **–disable-content-trust :** 忽略镜像的校验,默认开启

上传本地镜像mycentos:v1到镜像仓库中。

```shell
docker push mycentos:v1  
```

#### search

从docker Hub查找镜像。参数说明：

- **–automated :** 只列出 automated build类型的镜像；
- **–no-trunc :** 显示完整的镜像描述；
- **-f \<过滤条件>:** 列出指定条件的镜像。

从 docker Hub 查找所有镜像名包含 java，并且收藏数大于 10 的镜像  

```Python
docker search -f stars=10 java  
```

```shell
NAME                  DESCRIPTION                           STARS   OFFICIAL   AUTOMATED  
java                  Java is a concurrent, class-based...   1037    [OK]         
anapsix/alpine-java   Oracle Java 8 (and 7) with GLIBC ...   115                [OK]  
develar/java    
```

每列参数说明：

- **NAME:** 镜像仓库源的名称
- **DESCRIPTION:** 镜像的描述
- **OFFICIAL:** 是否 docker 官方发布
- **stars:** 类似 Github 里面的 star，表示点赞、喜欢的意思
- **AUTOMATED:** 自动构建
