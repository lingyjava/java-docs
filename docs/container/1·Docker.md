# 1·Docker

- [1·Docker](#1docker)
  - [相关命令](#相关命令)
  - [启动失败](#启动失败)
  - [安装PgSQL](#安装pgsql)
  - [安装MySQL](#安装mysql)
  - [安装nacos](#安装nacos)

## 相关命令
- [Docker 命令](https://www.runoob.com/docker/docker-ps-command.html)

启动容器mongo命名为mongodb，端口映射到27017
`docker run -name mongodb -p 27017:27017 mongo`

查看正在运行的容器
`docker ps `

通过id进入docker容器
`docker exec -it ad336fe4c0f8 bash`

## 启动失败
- [Error invoking remote method ‘docker-start-container‘: Error HTTP code 500](https://blog.csdn.net/SmallTeddy/article/details/121926669?spm=1001.2101.3001.6661.1&utm_medium=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1.pc_relevant_default&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1.pc_relevant_default&utm_relevant_index=1)

## 安装PgSQL
- [docker pgsql 镜像](https://hub.docker.com/_/postgres?tab=tags)
docker pull postgres:11.4
docker run --name postgres_demo -e POSTGRES_PASSWORD=password -p 54321:5432 -d postgres:11.4

run: 创建并运行一个容器；
--name: 指定创建的容器的名字；
-e POSTGRES_PASSWORD=password: 设置环境变量，指定数据库的登录口令为password；
-p 54321:5432: 端口映射将容器的5432端口映射到外部机器的54321端口；
-d postgres:11.4: 指定使用postgres:11.4作为镜像。

## 安装MySQL
- [数据库 | 彻底卸载MySQL8.0（win10）](https://www.jianshu.com/p/491f837e434b)
- [windows使用docker安装MySQL](https://www.jianshu.com/p/acc474171b63)

## 安装nacos
- [使用Docker完成Nacos集群部署](https://juejin.cn/post/6861996608247201806)

```java
docker run -d -e PREFER_HOST_MODE=hostname -e MODE=cluster -e NACOS_APPLICATION_POST=8846 -e NACOS_SERVERS="127.0.0.1:8846" -e SPRING_DATASOURCE_PLATFORM=mysql -e MYSQL_SERVICE_HOST=127.0.0.1 -e MYSQL_SERVICE_POST=3306 -e MYSQL_SERVICE_USER=root -e MYSQL_SERVICE_PASSWORD=123456 -e MYSQL_SERVICE_DB_NAME=nacos_config -e NACOS_SERVICE_IP=127.0.0.1 -p 8846:8846 --name nacos1 nacos/nacos-server
```