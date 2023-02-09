### Docker basics and interview

#### Docker简介
 Docker是一个开源的应用容器引擎，基于Go语言并遵从Apache2.0协议开源
 Docker是一个让开发者打包应用以及依赖到一个轻量级、可移植的容器中，然后发布到任何流行的Linux机器上，实现虚拟化
 容器完全使用`沙箱机制`，相互之间不会有任何接口，更重要的时容器性能开销极低

#### Docker的应用场景
 - Web应用的自动化打包和发布
 - 自动化测试和持续集成、发布
 - 在服务型环境中部署和调整数据库或其他的后台应用
 - 从头编译或扩展现有的OpenShift或Cloud Foundry平台搭建自己的Paas环境

#### Docker架构
 - 镜像(Image)：相当于一个root文件系统
 - 容器(Container)：镜像和容器的关系，就像面对对象编程设计中的类和实例一样，镜像时静态定义，容器是镜像运行时的实体
 - 仓库(Repository)：仓库可看作一个代码控制中心，用来保存镜像
 Docker使用的客户端-服务器(c/s)架构模式，使用远程API来管理和创建Docker容器
 Docker容器通过Docker镜像来创建
 - Docker客户端：通过命令行或其他工具与Docker守护进程通信
 - Docker主机：一个物理或者虚拟的机器用于执行Docker守护进程和容器
 - Docker Registry：通过标签指定镜像版本

#### Docker基础命令
 Docker的守护进程查看：`systemctl status docker`
 Docker镜像查看：`docker iamge ls`
 Docker容器查看：`docker ps / docker ps -a`
 Docker Registry配置和查看：`cat /etc/docker/daemon.json`
 查看docker相关进程：`ps -ef | grep docker`
 查看容器名称的命令：`docker ps --format "{{.Names}}"`
 在容器内部和宿主机中查看容器中的进程信息：`docker exec -it datermined_curie ps -ef`
 ...

#### Harbor
 - Harbor时VMware开源企业级Docker Registry项目，帮助用户迅速搭建一个企业级的Docker Registry服务
 - Harbor以Docker开源的Registry为基础，提供图形UI、基于用户访问控制`Role Based AccessControl`、AD/LDAP集成
 - Harbor的每个组件都是以Docker容器形式构建
 [更多参考](https://goharbor.io/)

#### Docker本地镜像载入与载出
 两种方法
 - 保存镜像(保存镜像载入后获得跟原镜像id相同的镜像)
 - 保存容器(保存镜像载入后获得跟原镜像id不同同的镜像)

#### Docker拉取镜像
 docker image pull / docker image pull xxx@v1.0.0

#### Docker保存镜像
 - docker save images'id -o /home/mysql.tar
 - docker save images'id > /home/mysql.tar
 ...

#### Docker载入镜像
 - docker load -i mysql.tar
 ...

#### Docker打个tag
 - docker tag xxx/xxx:v1.0.0
 ...

#### Docker Daemon原理
 作为Docker容器管理的守护进程，最初集成在命令中，到后来独立成单独二进制程序，其功能正在逐渐拆分细化，被分配到单独模块
##### 演进：Docker守护进程启动
 1.8以前守护进程启动命令：`docker -d`，该阶段守护进程只是Docker client的一个选项
 1.8开始启动命令：`docker daemon`，该阶段则是docker 命令的一个模块
 1.11开始，启动命令：`dockerd`，加上配置文件，和Docker client分离，独立成一个二进制程序

 ... (OCI / docker-shim / runc / docker-compose)

[docker官网文档](https://docs.docker.com/)
[更多参考](https://blog.csdn.net/crazymakercircle/article/details/128670335)

#### 总结
##### Docker改变什么
 - 面向产品：产品交付
 - 面向开发：简化环境配置
 - 面向测试：多版本测试
 - 面向运维：环境一致性
 - 面向架构：自动化扩容(微服务)
##### Docker的工作原理
 docker是一个C/S结构系统，docker守护进程运行在宿主机上
 守护进程从客户端接受命令并管理运行在主机上的容器
 ...