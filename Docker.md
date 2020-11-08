#### Docker

##### 一：概述

###### Docker为什么会出现

一款产品:开发–上线 两套环境 ->应用环境,应用配置.

开发 ---- 运维 问题:我在我的电脑上可以运行! 版本更新导致服务不可用,对于运维来说考验十分大

开发即运维!

环境配置十分麻烦,每个机器都要部署环境(集群Redis,ES,Hadoop…) 费时费力

发布一个项目 (jar + (Redis MySQL jdk ES)) ,项目能不能带上环境安装打包?

之前在服务器配置一个应用的环境Redis MySQL jdk ES Hadoop , 配置超麻烦,不能跨平台

Windows开发,发布到Linux上.

传统:开发打包成jar ,运维来做环境部署.

现在:开发打包部署上线,一套流程做完.

比如一个安卓应用 java – apk – 发布 (应用商店) --张三使用apk – 安装即可用

java – jar (环境) – 打包项目带上环境 (镜像) --(Docker仓库:商店) — 下载我们发布的镜像 – 直接运行即可

###### 比较Docker和虚拟机技术的不同

- 传统虚拟机，虚拟出一条硬件，运行一个 完整的操作系统，然后在这个系统上安装和运行软件-
- 容器内的应用直接运行在宿主机的内容， 容器是没有自己的内核的,也没有虚拟我们的硬件,所以就轻便了
- 每个容器间是相互隔离的,每个容器内部都有一个属于自己的文件系统,互不影响

##### 二：常用命令

###### 镜像命令

~~~shell
===============查看本机所有本地镜像=================
docker images
#解释
REPOSITORY  镜像的仓库源
TAG 		镜像的标签
IMAGE ID 	镜像的ID
CREATED		镜像的创建时间
SIZE		镜像的大小
# 命令的可选项
  -a, --all            列出所有镜像
  -q, --quiet          只显示镜像的ID
==================== 搜索镜像================================

docker search  镜像
#可选项 通过收藏来过滤s
--filter=STARS=3000 # 搜索的镜像就STARS大于3000以上的

==================下载镜像====================================
docker pull 镜像		//默认最新的


===================删除镜像===================================
docker rmi -f 镜像id
#批量删除
docker rmi -f $(docker images -aq)

~~~

###### 容器命令

~~~shell
=====================运行镜像======================
docker run  [可选参数] images
# 参数说明
--name="Name"  	容器名字  tomcat1 tomcat2 用了区分容器
-d  			后台方式运行
-it				使用交互方式运行,进入容器查看内容
#注意 p(小写)  P(大写)
-p				指定容器的端口   -p  8080:8080
		-p		ip:主机端口:容器端口
		-p  	主机端口: 容器端口(常用)
		-p  	容器端口
-P				指定随机端口	

#测试 启动并进入容器
[root@weifeng demo]# docker run -it centos /bin/bash
[root@211e4bb80345 /]# ls  # 查看内部容器的centos
bin  dev  etc  home  lib  lib64  lost+found  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
=======================退出镜像================
# 退出 后容器停止
[root@211e4bb80345 /]# exit
# 容器不停止退出
Ctrl+P+Q    

======================列出容器==================
#docker ps 命令
	# 当前正在运行的容器
-a  #列出当前正在运行的容器+带出历史运行过的容器	
-n=?#列出最近创建的容器	
-p  #只显示容器的编号
#列出运行的容器
[root@weifeng demo]# docker ps
#列出
[root@weifeng demo]# docker ps -a
#列出最近创建的容器 n后面数字可选
[root@weifeng demo]# docker ps -a -n=1
#只显示容器的编号
[root@weifeng demo]# docker ps -aq


========================删除容器=========================
#删除指定容器 运行中的不能删 如果强制删除 需要加 -f 参数(docker rm -f 容器id)
docker rm 容器ID 
#删除所有容器
docker rm -f $(docker ps -aq)

======================开始和停止容器=======================
docker start 容器id

docker restart 容器id

docker kill 容器id //强制停止容器id

docker stop 容器id

~~~

###### 其他命令

~~~shell
===========后台启动容器============
# 命令  docker run -d 镜像名
[root@weifeng demo]# docker run -d centos
c29b4906c418eb979e3dd111015508eca9e520cb08e6df51d5b41c5af2ab73c0
[root@weifeng demo]# docker ps
# 问题docker ps 发现 centos 停止了
# 常见的坑,docker 容器使用后台运行 必须要有一个前台进程,否则docker就会自动停止
==============查看容器日志================
docker logs 

-f 			:跟踪日志输出
--since 	:显示某个开始时间的所有日志
-t 			:显示时间戳
--tail 		:仅列出最新N条容器日志
#例子 运行容器 循环打印hello
[root@weifeng demo]# docker run -d centos /bin/bash -c "while true;do echo hello;sleep 1;done"

#显示日志 docker logs -tf --tail 10 容器ID
[root@weifeng demo]# docker logs -tf --tail 10 40123a87d3a1

==================查看容器中进程id===========


docker top 容器id


================查看容器元素据================


docker inspect 容器id


=================进入当前运行的容器================
#我们通常都是使用后台的方式运行容器,有时需要进入容器,修改一些配置
# 命令 方式一
 docker exec -it 容器id   bashShell 
# 方式二
[root@weifeng ~]# docker attach 容器ID
#两种区别
docker exec 	#进入容器后开启一个新的终端,可以在里面操作(常用)
docker attach   #进入容器正在执行的终端,不会启动新的进程


================从容器内部拷贝文件到主机上==================

docker cp 容器id:容器内文件路径   目的主机路径
#查看运行的容器
[root@weifeng ~]# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
c7c90ac66437        centos              "/bin/bash"         18 minutes ago      Up 39 seconds                           angry_cohen
#进入容器
[root@weifeng ~]# docker attach c7c90ac66437
[root@c7c90ac66437 home]# ls
my.java
[root@c7c90ac66437 home]# read escape sequence
#拷贝到/home文件下
[root@weifeng ~]# docker cp c7c90ac66437:/home/my.java /home

#拷贝式一个手动过程,之后可以通过 -v 卷的技术,可以实现文件自动同步

~~~

![img](Docker.assets/aHR0cHM6Ly9tZC1pbWFnZXNzLm9zcy1jbi1iZWlqaW5nLmFsaXl1bmNzLmNvbS9pbWFnZS0yMDIwMDUxNzA5MTk1Mzk2NC5wbmc)

![img](Docker.assets/aHR0cHM6Ly9tZC1pbWFnZXNzLm9zcy1jbi1iZWlqaW5nLmFsaXl1bmNzLmNvbS9pbWFnZS0yMDIwMDUxNzA5MjkyMzgxMi5wbmc)

![image-20200517092953977](Docker.assets/aHR0cHM6Ly9tZC1pbWFnZXNzLm9zcy1jbi1iZWlqaW5nLmFsaXl1bmNzLmNvbS9pbWFnZS0yMDIwMDUxNzA5Mjk1Mzk3Ny5wbmc)