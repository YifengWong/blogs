# Docker

## 一、 Ubuntu安装Docker 并修改镜像仓库源

1. 推荐使用阿里云，注册后到如下链接有简易教程
[https://cr.console.aliyun.com/?spm=5176.100239.blogcont29941.13.agXVRn#/accelerator](https://cr.console.aliyun.com/?spm=5176.100239.blogcont29941.13.agXVRn#/accelerator "阿里云Docker加速器")

2. 选择加速器，就会看到一行安装docker的快捷命令，需要安装curl
<pre lang="shell">
curl -sSL http://acs-public-mirror.oss-cn-hangzhou.aliyuncs.com/docker-engine/internet | sh -
</pre>

3. 配置文件 /etc/docker/daemon.json （如果没有可以自己新建）写入
<pre lang="shell">
# 其中XXX是你自己的加速镜像地址
{
  "registry-mirrors": ["https://XXXX.mirror.aliyuncs.com"]
}
</pre>

4. 重启docker，完成。
<pre lang="shell">
# systemctl为新ubuntu命令，可以控制服务
sudo systemctl daemon-reload
sudo systemctl restart docker
</pre>


## 二、 基本命令及参数使用
#### 1. 命令概览
> 容器生命周期管理
docker [run|start|stop|restart|kill|rm|pause|unpause]
容器操作运维
docker [ps|inspect|top|attach|events|logs|wait|export|port]
容器rootfs命令
docker [commit|cp|diff]
镜像仓库
docker [login|pull|push|search]
本地镜像管理
docker [images|rmi|tag|build|history|save|import]
其他命令
docker [info|version]

详见： 参考2

#### 2. run命令参数
<pre lang="shell">
docker run [OPTIONS] IMAGE [COMMAND] [ARG...] 
</pre>
详见： 参考3

常用OPTIONS:
- -d 后台运行，将返回容器ID
- --name="my-docker" 指定一个名称
- -e ENV_VAR="env_var" 设置环境变量
- -p host_port:docker_port 将docker的端口映射至主机端口
- -v host_path:docker_path 将主机目录挂载至docker中（Volume）

#### 3. Volume
> 为了能够保存(持久化)数据以及共享容器间的数据,Docker提出了Volume的概念.简单来说,Volume就是目录或者文件,它可以绕过默认的联合文件系统,而以正常的文件或者目录的形式存在于宿主机上.

删除Volume：如果你已经使用docker run来删除你的容器,那可能会有很多孤立的Volume仍在占用着空间
> Voulume可以被删除的条件:
1.该容器可以用docker rm -v来删除且没有其他容器连接到该Volume(以及主机目录是也没被指定为Volume).注意,-v是必不可少的.
2.docker run中使用rm参数.


## 三、 Dockerfile 使用
##### 1. Dockerfile可以简化使用过程，更便于配置、保存容器设置

##### 2. 书写规则
- Dockerfile的指令忽略大小写
- 使用#注释
- 每一行一条指令

##### 3. 构建指令：用于构建image，在运行前执行

##### 4. 设置指令：设置属性，在运行时执行

##### 5. 常用指令
<pre lang="shell">
# FROM 构建，作为第一条指令，指定基础image，可以位于远程或本地仓库image
# FROM centos:centos6
FROM <image>:<tag>


# MAINTAINER 构建，签名，写入个人信息
# MAINTAINER YifengWong "wyf951115@gmail.com"
MAINTAINER <name>


# RUN 构建，可以运行命令，进行安装程序或者设置属性等，有两种格式
# 命令内容语法应当是被基础镜像所支持的
# RUN apt-get install curl 
RUN <command>
RUN ["executable", "param1", "param2", ...]


# CMD 设置，指定容器运行时的操作，可以执行脚本等（也可以与ENTRYPOINT搭配使用）
# 该指令只能存在一个，若有多个，则执行最后一条
# CMD service tomcat7 start
CMD ["executable","param1","param2", ...]  
CMD command param1 param2


# ENTRYPOINT 设置，指定容器运行时的操作，可以执行脚本等
# 该指令只能存在一个，若有多个，则执行最后一条
# ENTRYPOINT service tomcat7 start
ENTRYPOINT ["executable","param1","param2", ...]  
ENTRYPOINT command param1 param2


# 如果存在多个CMD或ENTRYPOINT为完整命令，则互相覆盖，仅最后一条有效
# 搭配使用：
# CMD ["-l"]  
# ENTRYPOINT ["/usr/bin/ls"] 


# USER 设置，指定运行容器的用户
# USER root
USER username


# EXPOSE 设置，映射容器端口，可设置多个
# 如果运行容器时不指定-p参数到主机端口，则会随机映射到主机某端口
EXPOSE <port> [<port>...]


# ENV 设置，设置容器中的环境变量
# ENV JAVA_HOME /opt/jdk
ENV <key> <value>


# ADD 构建，可以复制文件或文件夹
# 将主机src复制到容器dest中，注意目录和文件的区别（最后是否有'/'）
# ADD和COPY类似，但是ADD会自动解压可识别的压缩文件
ADD <src> <dest>
COPY <src> <dest>


# VOLUME 设置，指定容器挂载点
# 如果运行容器时不指定-v参数到主机目录挂载，则会在主机随机生成目录挂载
VOLUME ["<mountpoint>"]


# WORKDIR 设置，切换目录
WORKDIR /path/to/workdir


</pre>


##### 6. 构建运行
<pre lang="shell">
# 构建，注意最后有一个点，指明当前目录下的dockerfile
docker build -t me/my-docker . 

# 运行容器
docker run -d -v /opt:/docker -p 8090:8080 me/my-docker 

</pre>


详见：参考4，5


## 参考
1. [https://yq.aliyun.com/articles/29941](https://yq.aliyun.com/articles/29941 "Docker 镜像加速器")
2. [http://blog.csdn.net/permike/article/details/51879578](http://blog.csdn.net/permike/article/details/51879578 "docker常用命令详解")
3. [http://blog.csdn.net/alen_xiaoxin/article/details/54694051](http://blog.csdn.net/alen_xiaoxin/article/details/54694051 "Docker run 命令参数及使用")
4. [https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/ "Best practices for writing Dockerfiles")
5. [https://docs.docker.com/engine/reference/builder/](https://docs.docker.com/engine/reference/builder/ "Dockerfile reference")