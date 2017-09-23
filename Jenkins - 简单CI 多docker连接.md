# Jenkins

## 结构
###### 比较简单作为示例，两个容器：Jenkins(test-jenkins) - MySQL(test-mysql)
###### Jenkins中运行项目，同时需要连接一个mysql数据库，使用两个并列关系的docker

## 准备
###### 需要在项目中，增加对docker数据库环境的配置，如jdbc连接
<pre lang="shell">
# 连接的数据库主机名在后面运行Jenkins容器时指定docker_mysql
db.jdbcUrl=jdbc:mysql://docker_mysql:3306/yizhu_test?createDatabaseIfNotExist=true&useSSL=false
</pre>

## 安装

###### 1. MySQL - Dockerfile内容如下：

<pre lang="shell">
# 使用Dockerfile可以方便执行更多的自定义操作
# 从mysql的官方5.7版本获取
FROM mysql:5.7

# 签名
MAINTAINER YifengWong "wyf951115@gmail.com"

# 通过设置环境变量，设置root用户的密码
ENV MYSQL_ROOT_PASSWORD 123456

# 映射容器的3306端口
EXPOSE 3306

</pre>

###### 2. 在Dockerfile所在目录运行
<pre lang="shell">
# build docker，命名为wyfdocker/mysql 注意最后有个点，指定目录，意为当前目录
sudo docker build -t wyfdocker/mysql .

# 运行容器，命名为test-mysql，将容器3306端口映射至宿主机3307端口
sudo docker run --name test-mysql -p 3307:3306 -d wyfdocker/mysql

</pre>

###### 3. Jenkins
<pre lang="shell">
# --name myjenkins 命名为自定义
# --link 连接docker，可以将名为test-mysql的docker容器使得在jenkins中以docker_mysql作为机器名访问，通过如此连接各个容器是docker推荐的方式
# -p 端口映射，将容器的端口（后者）映射至主机端口（前者）
sudo docker run --name myjenkins --link=test-mysql:docker_mysql -p 8080:8080 -p 50000:50000 -v /home/craze/docker/jenkins_data:/var/jenkins_home jenkins

</pre>

###### 4. 启动过程中会显示一个秘钥，如下，登录进jenkins后台时需要使用
<pre lang="shell">
Jenkins initial setup is required. An admin user has been created and a password generated.
Please use the following password to proceed to installation:

dad060c0016f42dfb646f4d6e0ac5b01

</pre>

## 进入Jenkins


浏览器进入 localhost:8080 
即需要输入该秘钥，随后安装建议安装的插件，并新建用户
![](http://119.29.166.163/wp-content/uploads/2017/06/jenkins-01.png)

### 开始新任务（Maven - GitHub 持续集成）
1. 系统管理-插件管理 中 下载安装 Maven Integration plugin

2. 再到系统管理-Global Tool Configuration中增加maven安装（自动or解压包等等方式）
![](http://119.29.166.163/wp-content/uploads/2017/06/jenkins-02.png)

3. 构建一个新项目，如我的实训易助平台（yizhu）
![](http://119.29.166.163/wp-content/uploads/2017/06/jenkins-03.png)

4. 选择GitHub project，并填写GitHub项目URL

5. 源码管理选择Git，并填写仓库URL，需要添加账号或者使用SSH

6. 构建触发器，使用脚本，或使用GitHub Hook触发器（需要额外插件以及github配置）

7. 构建选择 增加构建步骤 Invoke top-level Maven targets
![](http://119.29.166.163/wp-content/uploads/2017/06/jenkins-04.png)
一般需要执行单元测试，可以使用如下mvn命令
<pre lang="shell">
clean install test
</pre>

8. 点击保存，左面板即可“立即构建”，完成。


## 参考
1. Jenkins安装
[https://docs.docker.com/samples/jenkins](https://docs.docker.com/samples/jenkins "Jenkins安装")
2. Docker Volume容器 [http://blog.csdn.net/shanyongxu/article/details/51460930](http://blog.csdn.net/shanyongxu/article/details/51460930 "Docker Volume容器")