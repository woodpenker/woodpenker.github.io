---
title: docker 学习笔记
tags: [linux, docker]
categories: linux
---
<h1>docker 学习笔记</h1>

概念理解
---
docker的功能比较像一个chroot应用,拥有一个守护进程daemon.

docker容器并不是一个完整的系统,不包含linux内核,并且只是一个最小系统.docker容器的内核依旧是宿主系统的内核.docker利用了linux内核的**命名空间namespace**,**容器组cgroup**,**设备映射**的技术.
<!--more-->
docker的命令使用起来比较像git.

docker拥有官方hub和可以私有搭建的镜像库registry(默认docker官方的docker hub),提供镜像下载服务.

docker文件系统利用了**联合加载(union mount)**和**写时复制(copy on write)**的技术(aufs也是该技术),可以在文件系统上叠加文件结构,形成层级关系,但是从纵向看过去就像一个文件系统.通过镜像库下载的镜像作为最底层rootfs,并不能被修改,而是每次修改都记录在一个层中,在最顶层加载一个读写文件系统,使得看上去好像我们修改了底层系统一样.

docker目前使用时必须具有**root权限**.因为需要用到宿主底层的接口.

docker启动后会修改镜像中的`/etc/hosts`和`/etc/resolv.conf`文件,使得docker容器可以使用主机的网络环境,以及记录docker network子网路由解析信息等.docker会感知所使用网络下的所有容器并将相应的DNS信息写入每个容器的`/etc/hosts`中,如果指定了容器的名称,主机可以使用hostname.netname的形式解析DNS.

docker容器默认不对外公开任何接口,是无法访问的,需要给容器指定对外的接口

docker默认使用**172.17.x.x**作为子网地址,除非该子网被占用.如果占用,则从**172.16~172.30**中尝试.

docker安装时会创建一个新的网络接口**docker0**,作为默认网络桥.它是一个虚拟的以太网桥,用于连接宿主和容器.docker每创建一个容器就会创建一组互联的网络接口,一端作为容器的eth0接口,一端作为veth开头的接口存在与宿主作为一个宿主端口,可以认为它是一根网线连接在docker0和容器内网卡上.

(1.9版本以上)可以创建自己的专用容器间网络,称为**Docker Networking**.使得容器可以跨越不同的主机进行通信,并且是支持可插拔的.

docker容器间通讯的两种方式: 通过docker网络 和 使用link链接连接容器.

docker常用命令
---
**守护进程/usr/bin/docker启动选项**
```
--log-driver 下同docker run的 --log-driver,版本1.6以后可用
-insecure-registry localhost:5000 //启动守护进程时指定本地registry
--ice=false 关闭所有没有链接的容器间的通讯
-H tcp://0.0.0.0:2375 将docker守护进程绑定到宿主机的所有网络的2375端口上DOCKER_HOST环境变量代指该参数
-d 作为daemon守护进程启动在后台
```

**启动基于容器的本地registry**
```
sudo docker run -p 5000:5000 registry:2 
```

**docker run** 创建并运行一个容器
```
sudo docker run -i -t ubuntu:16.04 /bin/bash //镜像名ubuntu后加:16.04指定ubuntu仓库类目下的某一镜像16.04,/bin/bash是启动容器后需要执行的命令

-i 保证stdin开启,以便支持交互式访问容器
-t 为容器分配一个伪tty终端,以便交互式访问容器
-d 将容器放在后台运行,使用deamon模式
-p 8080:80 指定docker容器运行时对外公开的端口,将容器的80端口映射到宿主机的8080端口,
-p 80 则docker在宿主机中从32768-61000中选择一个端口号映射到容器的80端口
-p 127.0.0.1:8080:80/udp 则指定了绑定的本地宿主机的IP,并且指定UDP端口.
-P 对外公开在Dockerfile中使用EXPOSE指令公开的所有端口
--name 为启动的容器指定一个名称,即容器名,而不是使用自动生成的名称,支持a-z,A-Z,0-9,下划线,圆点,横线,即正则:[a-zA-Z0-9_.-]
--log-driver 控制容器使用日志驱动,可选:syslog(禁用docker logs并将所有容器日志重定向到syslog中),默认json-file,none(禁用日志)等
--restart=always/on-failure:5 让容器自动重启,always是一直重启,on-failure指定容器退出代码非0后重启,后面数字指定当容器退出代码非0以后重启的次数.(1.2版本以后)
--entrypoint 标志可以覆盖ENTRYPOINT指令
-w 指令会在运行时覆盖WORKDIR工作目录设置 -w /var/log
-e 传递环境变量 -e "WEB=/var/www/html"
-u 覆盖USER指令
--build-arg 开启ARG参数,其后可以给参数指定值,如 --build-arg build=1234
-v 指定一个目录映射对应于容器内的一个目录,将宿主机中的目录作为卷挂载到容器,使用:分割,后面加ro(只读)或rw(可读写)指定读写权限,如果容器内目录不存在则自动创建一个. 如: -v /home/web:/var/www/html:ro
--net=mynet 指定运行时使用的docker网络
--link A:B 创建两个容器间的C-S链接,A是要链接的容器名,B是链接的别名.--link链接的容器不需要对外公开接口.可以多处使用指定多个容器.docker会在父容器修改/etc/hosts和写入包含链接信息的环境变量(以别名B的大写开头).
--hostname 或-h 指定容器的主机名,容器内主机的名称
--add-host 在/etc/hosts中添加主机记录
--dns或--dns-search为某个容器单独配置DNS(写入/etc/resolv.conf).
--privileged 启动docker特权模式,允许以宿主机具有的(几乎)所有能力来运行容器,包括内核特性和设备访问,用来在docker中运行docker.容器在这个模式下对宿主具有root访问权限.
--cidfile=/tmp/container.id.txt,让docker截获容器ID并存到指定的文件里.
--volume-from ubuntu-test 把指定容器ubuntu-test里的所有卷都加入到新创建的容器里,即便ubuntu-test没有运行,也能访问到它的卷.即使删除了使用卷的最后一个容器,卷中的数据也会持久保存.
--rm 标志创建只用一次的容器,使用完就删除.可以用于测试或者备份容器内的卷:

sudo docker run --rm --volumes-from ubuntu-test -v /home:/backup ubuntu tar cvf /backup/bk.tar /data  
//使用ubuntu基础镜像创建一个只用一次的镜像,将宿主本地/home目录挂载到ubuntu-test容器的/backup下,并且执行压缩命令将/data目录下的文件压缩到/backup/bk.tar文件中.这样在宿主的目录下就会有一个bk.tar作为ubuntu-test下的/data的备份.

```

**docker ps** 列出正在运行的容器
```
-a 列出所有容器
-q 只返回容器的ID列不返回其他信息
```

**docker start** 启动已经创建并停止的容器
```
sudo docker start ubuntu_test
```

**docker attach** 重新附着到重启以后的容器会话上
```
sudo docker attach ubuntu_test
```

**docker logs** 获取容器的日志
```
sudo docker logs ubuntu_test

-f 持续监控容器日志,类似tail -f,crtl+c退出监控
-t 为每条日志加上时间戳,便于调试
```

**docker top** 查看以守护式方式运行在后台的容器内的进程
```
sudo docker top ubuntu_test
```

**docker exec** 在正在运行容器内部执行新的命令或额外启动新的进程(1.3版本以后)
```
sudo docker exec -d ubuntu_test touch 1.file

-d 指定要在内部执行命令的容器的名字和执行的命令
-u 为新启动的进程指定一个用户属性(1.7版本以后)
```

**docker stop** 停止守护式的容器
```
sudo docker stop c2c4e57c12
```

**docker kill** 发送信号给容器
```
sudo docker kill -s <sibnal> <container>
```

**docker inspect** 获取容器的详细信息
```
sudo docker inspect ubuntu_test
-f/--format={{ }} 筛选结果,支持使用go的模板
sudo docker inspect --format='{{ .Config.Hostname }}' ubuntu_test 只显示出Config下Hostname子项的内容
```

**docker rm** 删除不再使用的容器
```
sudo docker rm 80430f8d
-f 强制删除正在运行的容器
sudo docker rm $(sudo docker ps -a -q) 组合命令可以删除所有容器
```

**docker rmi** 删除一个或多个镜像
```
sudo docker rmi ubuntu_test test //删除多个镜像
sudo docker rmi $(sudo docker images -a -q) //删除所有镜像
```

**docker images** 列出docker镜像

本地的镜像都保存在`/var/lib/docker`下,所有容器在`/var/lib/docker/container`下

**docker pull** 将某个镜像从registry中拉取下来到本地

`sudo docker pull fedora:20`

**docker search** 在registry中查询某个镜像

`sudo docker search ubuntu`

**docker commit** 提交某个镜像
```
sudo docker commit -m "new image" -a "my" 433b23ce mydockeraccount/test:webser //将433b23ce这个镜像提交到docker hub中我的账号下名为test的仓库类目下并且使用:指定了镜像标签为webser

-m 指定新创建的镜像的提交信息
-a 指定该镜像的作者信息 

```

**docker build** 使用Dockerfile文件构建镜像
```
可以使用 镜像名:标签 的格式可以设置镜像标签,如果没有指定标签自动设置latest标签
-t 为新构建镜像设置仓库和名称
-f 指定区别于默认的位置的Dockerfile,如 -f path/to/file,这个位置相对于构建环境上下文目录.
--no-cache 关闭缓存功能,每次构建都重头开始,不使用缓存.
```

**docker login** 登录doker hub

`sudo docker login`登录信息记录在`$HOME/.dockercfg`或`$HOME/.docker/config.json(1.7版本以上)`中

**docker logout** 退出registry

**docker history** 查看镜像构建的每一步都做了什么,是如何构建出来的

`sudo docker history 22d48fg //列出该镜像构建时的每一层和Dockerfile指令`

**docker port** 查看容器的端口映射情况

`sudo docker port ubuntu-test 80 //查看ubuntu-test的80端口映射到宿主哪个端口`

**docker network** docker networking 相关操作
```
sudo docker network create mynet //创建名为mynet的docker桥接网络,命令返回该网络的ID.创建overlay网络会允许我们在多宿主间通信.

sudo docker network inspect mynet //查看创建的docker网络

sudo docker network ls //列出当前系统中的所有docker网络

sudo docker network connect mynet ubuntu-test//添加正在运行的容器ubuntu-test到已有的网络mynet中

sudo docker network disconnect mynet ubuntu-test//从网络mynet中断开容器ubuntu-test

docker run 时可以指定容器允许所使用的网络
```




Dockerfile文件
---
Dockerfile使用DSL语法,所有指令都必须为大写.

Dockerfile是顺序执行的.

在构建环境(上下文)目录(即使用当前所处在的目录位置为构建环境)下,新建名为Dockerfile的文件.Dockerfile大致执行流程为:从基础镜像运行一个容器,每次执行一个指令修改并提交操作生成新的镜像层,在该镜像层基础上运行新的容器,再重复执行下一条指令.某一指令执行失败可以使用上一指令执行后的容器名称进行调试.

docker构建每一步都会提交为镜像,就使得之前构建过程中的镜像可以被当作缓存,再次执行构建操作时如果构建操作没有变化会略过构建而直接使用缓存.

构建环境下.dockeignore文件可以指定目录下那些文件不被当作构建环境中的上下文的一部分,类似git的gitignore.该文件使用go的filepath中的匹配规则.

每个Dockerfile的第一条指令必须是FROM.

**RUN** 
```
指定镜像被构建时运行的命令

RUN pip install -r requirement.txt //执行pip命令从requirement.txt文件中读取需要安装的依赖名并安装.
RUN会在shell中默认使用`/bin/sh -c `作为执行命令的前缀

推荐使用数组格式的RUN指令: RUN [ "apt-get","install","-y","nginx" ],数组语法
```

**CMD**
```
指定容器启动时要运行的命令

CMD [ "/bin/bash", "-l" ],使用数组语法,不指定时docker会在命令前加`/bin/sh -c`(-c 表示使用后续字符串作为命令而不是从标准输入读取命令),推荐使用数组语法. 
docker run可以覆盖CMD指令.
```

**ENTRYPOINT**
```
指定在运行时要运行的命令,且不会轻易被run指令覆盖.

docker run指令中的参数被作为该命令的参数传递给ENTRYPOINT中的指令.
可以同时使用CMD和ENTRYPOINT,指定启动时的默认行为:
ENTRYPOINT ["/usr/bin/nginx"]
CMD ["-g","daemon off"]
指定默认启动nginx并让nginx守护进程以前台方式运行(避免容器自动退出),docker run 命令以其他参数 如 -h 覆盖该指令行为.
docker run 的--entrypoint 标志可以覆盖ENTRYPOINT
```

**WORKDIR**
```
指定一个工作目录,后续的ENTRYPOINT和CMD指令会在这个目录下执行.

WORKDIR /home
docker run 的-w指令会在运行时覆盖工作目录
```

**ENV**
```
指定镜像构建过程中的环境变量.

这些变量会永久保留到从这个镜像创建的任何容器.
docker run 使用-e 传递环境变量
ENV WWW /var/www/html //指定WWW变量代表/var/www/html
WORKDIR $WWW //切换工作目录到/var/www/html
```

**USER**
```
指定镜像以什么用户运行

USER uid:gid
USER user 
USER group 
//指定用户和组或用户或组,不指定则默认使用root用户.
docker run 使用-u 覆盖该指令
```

**VOLUME**
```
指定一个目录位置,用来向基于镜像创建的容器添加卷

VOLUME ["/var/www/html"]
一个卷可以存在与多个容器内的特定目录,即可以共享,这个特定的目录会绕过联合文件系统.
一个容器可以不和其他容器共享卷
对卷的修改是立即生效的
对卷的修改不会对镜像产生影响
卷会一直存在直到没有任何容器再使用它.
卷可以方便向镜像中添加数据代码等而不用提交新的镜像.
可以使用数组方式指定多个卷VOLUME ["/var/www/html","/tmp"]
```

**ADD**
```
用来将构建环境下的文件和目录或URL位置对应的资源复制到镜像中.

ADD subdir/test.txt /home/test/
不能对构建环境以外的文件进行操作.
docker通过地址的结尾字符来判断目标位置是文件还是目录,以/结尾的均看做目录.
ADD对于gzip,bzip,xz源文件会自动解压,与tar -x功能类似.如果目的地址已经存在同名文件或目录不会覆盖.如果目的位置不存在会创建全路径(包含路径中任何目录),新创建的文件和目录mod为0755,gid和uid为0.
ADD指令会使得构建缓存机制变得无效.
```

**COPY**
```
类似ADD,COPY只在构建上下文中复制本地文件,不做提取和解压工作.
目的地址必须是容器内的绝对路径.
```

**LABEL**
```
(1.6版本以上)为docker镜像添加元数据,元数据以键值对形式出现,推荐所有元数据放在一列,以免创建多层镜像.

LABEL ver="1.1" type="date"
```

**STOPSIGNAL**
```
(1.9版本以上)设置停止容器时发送的系统调用信号,必须是内核系统调用表中的合法的数,如 SIGKILL,9 
```

**ARG**
```
(1.9版本以上)定义可以在docker build 命令运行时传递给构建的变量.构建时只需使用--build-arg标志即可.

ARG build //未指定build参数的默认值
ARG target=/var/www/html //指定了target参数的默认值
docker预定义的ARG变量,无需在Dockerfile中定义:
HTTP_PROXY
http_proxy
HTTPS_PROXY
https_proxy
FTP_PROXY
ftp_proxy
NO_PROXY
no_proxy
```

**ONBUILD**
```
为镜像添加触发器,当一个镜像被用作其他镜像构建的基础镜像时会触发执行,在构建过程中插入新的指令.相当于这些指令紧跟在构建其他镜像的Dockerfile的FROM后面.

ONBUILD ADD . /app/
ONBUILD会顺序执行,并且只能被继承一次,不会出现在孙子镜像中.
FROM,MAINTAINER,ONBUILD不能被用在ONBUILD中,以防递归.
```


Remote API
---
docker守护进程提供remote api, 默认情况下会在宿主创建套接字`unix:///var/run/docker.sock`

`echo -e "GET /info HTTP/1.0\r\n" |sudo nc -U /var/run/dcoker.sock // 查询本地docker API `

要启动docker API 需要使用-H给守护进程传递参数,可以修改守护进程的启动配置文件,将守护进程永久绑定到指定的网络接口上.
```
Ubuntu/Debain:	/etc/default/dcoker
RedHat/Fedora:	/etc/sysconfig/docker
使用systemd的版本:	/usr/lib/systemd/system/docker.service
```
可以使用POST请求来调用/container/create接入点来创建容器,等同于`docker run`:
```
curl -X POST -H "Content-Type: application/json" http://localhost:2375/container/create -d '{"Image":"ubuntu-test"}'
```

docker官网有[docker Remote API客户端库](http://docs.docker.com/reference/api/remote_api_client_libraries/)的完整列表.

Docker Remote API 可以使用TLS/SSL进行认证,确保安全.需要自己创建CA证书.
