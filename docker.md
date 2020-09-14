# **docker**

## **安装docker**

### 环境要求

* 系统内核3.10以上 :

  ```shell
  uname -r
  
  #3.10.0-1127.19.1.el7.x86_64
  ```

* centos7以上

  ```shell
  cat /etc/os-release 
  ```

### 安装步骤

- 卸载旧版本
```shell
  $ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

- 需要的安装包
  ```shell
  sudo yum install -y yum-utils
  ```

- 设置镜像仓库（阿里云） 

```shell
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

- 更新软件安装包的索引
```shell
yum makecache fast
```
- 安装
```shell
sudo yum install docker-ce docker-ce-cli containerd.io
```
- 启动
```shell
sudo systemctl start docker
```
- 测试shell 
```shell
docker version 
docker run hello-world
```
- 配置阿里云镜像加速器

- 卸载docker
```shell
$ sudo yum remove docker-ce docker-ce-cli containerd.io
$ sudo rm -rf /var/lib/docker
```
## 镜像命令

- 查看镜像

 ```shell
docker images

docker images --help

docker images -a #显示所有

docker images -q #只显示id
 ```

- 查找镜像

```shell
docker search imagename
```

- 下载镜像

```shell
docker pull mysql #默认下载最新

docker pull mysql:5.7
```

- 删除镜像

```shell
docker rmi -f imageid

docker rmi -f $(docker images -aq)
```

## 容器命令

- 启动

```shell
docker run [可选参数] image

--name Name #设置容器名字

# docker run -d --name nginx01 -p 3344:80 nginx

# -d 后台方式运行

-it #使用交互方式运行，进入容器查看内容

#如：docker run -it centos /bin/bash  exit退出容器

-p #指定容器的端口 -p 8080:8080

-p ip:主机端口:容器端口

-p 主机端口:容器端口

-p 容器端口

-P 随机指定端口

--rm 用完即删,docker ps -a 都看不到
	#docker run -it --rm tomcat:9.0
	#docker run -d --rm -p 8080:8080 tomcat:9.0
-e 环境配置
```

- 查看运行

```shell
#当前在运行的容器
docker ps 

#历史运行的容器
docker ps -a

#显示指定数量的容器
docker ps -a -n=2
docker ps -n=3

#只显示编号
docker ps -q
```

- 进入容器

  方式1：进入当前正在运行的容器

```shell
docker exec -it containerid /bin/bash
```

 	  方式2 :  进入容器后开启一个新的终端。

``` shell
docker attach containerid #进入容器正在执行的终端，不会启动新的进程！
```

- 退出容器

```shell
exit #直接容器停止并退出

#ctrl+p+q 不停止容器退出
```

- 删除容器

```shell
docker rm 容器id  #不能删除运行中的

docker rm -f 容器id

docker rm -f $(docker ps -aq) #强制删除

docker ps -aq |xargs docker rm #通过管道删除
```

- 启动和停止容器

```shell
docker start 容器id

docker restart 容器id

docker stop 容器id

docker kill 容器id #强制停止容器
```

## 常用其他命令

- 后台启动容器

```shell
docker run -d image
```

问题：docker ps 发现停止了。

常见的坑：容器使用后台启动，就必须有一个前台进程，docker发现没有应用就会自动停止，比如nginx。

解决思路：启动后，创建进程

```shell
docker run -d centos /bin/sh -c "while true;do echo hello hi;sleep 1;done" #就不会停止了。
```

- 动态查看日志

```shell
docker logs -tf containerid

docker logs -tf --tail 10 containerid  #查看历史10条开始
```

- 查看镜像的元数据

```shell
docker inspect containerid
```

- 从容器内拷贝文件到主机上

```shell
docker cp containerid:/home/test.java /home #容器可以没有运行中。
```

- 查看内存状态

```shell
docker stats 
```

## 可视化

- portainer（不怎么会用到）

```shell
docker run -d -p 8088:9000 --restart=always -v /var/run/docker.sock:/var/run/docker.sock --privileged=true portainer/portainer #然后直接访问即可
```

- rancher（CI/CD）

## docker的联合文件系统

- 分层，分层下载

- commit镜像

  实战测试：1.启动一个默认的tomcat，2.发现默认的没有webapps引用。3.自己拷贝。4.然后我们commit提交一个镜像，以后我们就使用修改后的镜像。

```shell
docker commit -a="zxd" -m="add webapps app" containerId mytomcat01:1.0
```

## 容器数据卷

- -v 挂载（双向同步）：主机目录：容器目录

```shell
docker run -it -v /home/ceshi:/home centos /bin/bash
```

```shell
#测试：安装mysql
docker run --name mysql01 -p 3310:3306 -v /home/mysql/conf:/etc/mysql/conf.d -v /home/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=root -d mysql:5.7
```

- 匿名挂载：在默认路径下创建一串英文作为容器挂载路径

   -v /etc/nginx

- 具名挂载 ：在默认路径下创建指定名字作为容器挂载路径

  -v juming-ng:/etc/nginx

- 查看： 
  ```shell
  docker volume ls
  ```
  
- 查看具体信息 
  
  ```shell
    docker volume inspect volume_name
  ```
  
- 默认挂载卷的位置"Mountpoint": "/var/lib/docker/volumes/volume_name/_data"

- 扩展：-v /home/ceshi:/home:ro 什么意思

  ```
  ro：read only 只读 ：只能在宿主机中修改
  
  rw：read write 可读可写
  ```


## Dockerfile

> 构建docker镜像的构建文件。

- 快速开始

  ```shell
  #编写dockerfile1
  vim dockerfile1
  
    FROM centos                   
  
    VOLUME ["volume01","volume02"]  //匿名挂载
  
    CMD echo "-----end-----"      
  
    CMD /bin/bash 
  #构建 -f file -t target
   docker build -f /home/docker_test_volume/dockerfile1 -t zxd/centos:1.0 .    
  
  ```

- 编写Dockerfile

  ![img](https://qqadapt.qpic.cn/txdocpic/0/7ce0fec2ae0e3dba9872ce3279feb7ff/0?w=904&h=322)

 ```shell
#练习：因为默认的centos没有vim pwd ifconfig命令 我们创建一个image引入这些功能
#dockerfile：

[root@localhost dockerfiles]# cat mydockerfile-centos

FROM centos

MAINTAINER zxd<1726211067@qq.com>

ENV MYPATH /usr/local

WORKDIR $MYPATH

RUN yum -y install vim

RUN yum -y install net-tools

EXPOSE 80

CMD echo $MYPATH

CMD echo "---end---"

CMD /bin/bash

#构建：

 docker build -f mydockerfile-centos -t zxdcentos:0.1 .
#运行
 docker run -it zxdcentos:0.1 

#我们可以用下面查看其它镜像是怎么构建的

docker history [OPTIONS] IMAGE
 ```

- CMD和ENTRYPOINT区别

```shell
[root@localhost dockerfiles]# cat dockerfile-cmd

FROM centos

CMD ["ls","-a"]

docker run 243b07ea6534 #可以打印

docker run 243b07ea6534 ls -a #可以打印

docker run 243b07ea6534 -l #报错

FROM centos

ENTRYPOINT ["ls","-a"]

docker run 243b07ea6534 #可以打印

docker run 243b07ea6534 -l #可以打印
```

- 制作tomcat的镜像

     ![img](https://qqadapt.qpic.cn/txdocpic/0/400ce7d4d26db6e784200bc1e2f99b9b/0?w=1302&h=539)            

构建如果是Dockerfile名字不用写 -f

启动      ![img](https://qqadapt.qpic.cn/txdocpic/0/dc7741566209b557735a8f78c28115f4/0?w=1421&h=107)            

## 数据卷容器

实现多个容器共享（相互拷贝）

```shell
--volumes-from #容器id或者名字

docker run --name centos22 --volumes-from d3e4b745b1d0 -it zxd/centos:1.0
```

## 发布镜像

- 发布镜像到dockerHub

​            ![img](https://qqadapt.qpic.cn/txdocpic/0/014fa3e31cecc13e7fd42fc56b5d2b39/0?w=907&h=591)            

如果出问题 push不了

​            ![img](https://qqadapt.qpic.cn/txdocpic/0/9c488b67e966933d94a73cb5cf93a693/0?w=754&h=77)            

- 发布镜像到阿里云 to容器镜像服务

小结：            ![img](https://qqadapt.qpic.cn/txdocpic/0/5048198a86a4d946191d0a30610f614a/0?w=777&h=608)            

## docker网络

​            ![img](https://qqadapt.qpic.cn/txdocpic/0/1955e27e48fa1fd0753c5009fc2d5ca5/0?w=1025&h=503)            

有本机回环地址、阿里云内网地址、docker0地址

docker的桥接模式，网卡成对出现veth

​            ![img](https://qqadapt.qpic.cn/txdocpic/0/54fdf9bdc388bc22cccc60242d6d4ee5/0?w=988&h=646)            

### --link

- Quick start

```shell
--link

docker run -d -P --name tomcat01 tomcat

docker run -d -P --name tomcat02 tomcat

docker exec -it tomcat02 ping tomcat01 #ping不成功

ping: tomcat01: Name or service not known

docker run -d -P --name tomcat03 --link tomcat01  tomcat

docker exec -it tomcat03 ping tomcat01 #成功
```

- 原理

```shell
[zxd997@localhost dockerfiles]$ docker exec -it tomcat03 cat /etc/hosts

127.0.0.1       localhost

::1     localhost ip6-localhost ip6-loopback

fe00::0 ip6-localnet

ff00::0 ip6-mcastprefix

ff02::1 ip6-allnodes

ff02::2 ip6-allrouters

172.17.0.2      tomcat01 dfb824443030

172.17.0.3      fffca5ffb2a2
```

所以反向是不行的。

**不建议使用--link，自定义网络不适用docker0 ，docker0不支持容器名连接访问**

### 自定义网络

- 查看所有网络

  docker network ls 

- 网络模式：

bridge：桥接模式 docker默认，自己创建也使用bridge模式

none：不配置网络

host：和宿主机共享网络

container：容器网络连通（用的少，局限很大）

- --net 

不写默认是 --net bridge

docker0的特点：默认，域名不能访问，--link可以打通连接

```shell
#我们可以自定义一个网络

docker network create --driver bridge --subnet 192.168.0.0/16 --gateway 192.168.0.1 mynet

#然后启动两个tomcat，使用刚刚创建的网络

docker run -d -P --name tomcat11 --net mynet tomcat

docker run -d -P --name tomcat22 --net mynet tomcat

docker network inspect mynet #会看到container多了两个网络

#好处：不用--link，也能ping name了
```

### 网络连接

连通不同网络下的容器

```shell
docker run -d -P --name tomcat4 tomcat

docker exec -it tomcat4 ping tomcat11

ping: tomcat11: Name or service not known #ping不通

docker network connect mynet tomcat4 #就是把mynet网络加了tomcat4，

docker exec -it tomcat4 ping tomcat11 #成功

#所以tomcat4就能ping通mynet下的容器了
```

## 搭建redis集群

分片+高可用+负载均衡

```shell
#创建redis集群网络

docker network create --subnet 192.169.0.0/16 redis

#shell创建6个redis配置文件：

for port in $(seq 1 6); \

do \

mkdir -p /mydata/redis/node-${port}/conf

touch /mydata/redis/node-${port}/conf/redis.conf

cat << EOF >/mydata/redis/node-${port}/conf/redis.conf

port 6379

bind 0.0.0.0

cluster-enabled yes

cluster-config-file nodes.conf

cluster-node-timeout 5000

cluster-announce-ip 192.169.0.1${port}

cluster-announce-port 6379

cluster-announce-bus-port 16379

appendonly yes

EOF

done

#shell 启动6个redis

for port in $(seq 1 6); \

do \

docker run -p 637${port}:6379 -p 1637${port}:16379 --name redis-${port} \

-v /mydata/redis/node-${port}/data:/data \

-v /mydata/redis/node-${port}/conf/redis.conf:/etc/redis/redis.conf \

-d --net redis --ip 192.169.0.1${port} redis:5.0.9-alpine3.11 redis-server /etc/redis/redis.conf; \

done

#创建集群

 redis-cli --cluster create 192.169.0.11:6379 192.169.0.12:6379 192.169.0.13:6379 192.169.0.14:6379 192.169.0.15:6379 192.169.0.16:6379 --cluster-replicas 1

#打印信息默认分配3master3slave

Master[0] -> Slots 0 - 5460

Master[1] -> Slots 5461 - 10922

Master[2] -> Slots 10923 - 16383

Adding replica 192.169.0.15:6379 to 192.169.0.11:6379

Adding replica 192.169.0.16:6379 to 192.169.0.12:6379

Adding replica 192.169.0.14:6379 to 192.169.0.13:6379

M: 7f4666840dd557177d0b9a2e38f37980e2399aae 192.169.0.11:6379

   slots:[0-5460] (5461 slots) master

M: 5a6bafeef212cab67598403a006cb12e033ed2a3 192.169.0.12:6379

   slots:[5461-10922] (5462 slots) master

M: bfeb82d5509cba9576942dafda595a56e2e247b8 192.169.0.13:6379

   slots:[10923-16383] (5461 slots) master

S: 4323af513da7a72974ef948b25fc4093054b460f 192.169.0.14:6379

   replicates bfeb82d5509cba9576942dafda595a56e2e247b8

S: a57402b9a5f7de01a85b47d85fe540efbaa277c7 192.169.0.15:6379

   replicates 7f4666840dd557177d0b9a2e38f37980e2399aae

S: 7c25854b274173ef77847143250ba12d76918f40 192.169.0.16:6379

   replicates 5a6bafeef212cab67598403a006cb12e033ed2a3

Can I set the above configuration? (type 'yes' to accept): yes

\>>> Nodes configuration updated

\>>> Assign a different config epoch to each node

\>>> Sending CLUSTER MEET messages to join the cluster

Waiting for the cluster to join

...

\>>> Performing Cluster Check (using node 192.169.0.11:6379)

M: 7f4666840dd557177d0b9a2e38f37980e2399aae 192.169.0.11:6379

   slots:[0-5460] (5461 slots) master

   1 additional replica(s)

M: bfeb82d5509cba9576942dafda595a56e2e247b8 192.169.0.13:6379

   slots:[10923-16383] (5461 slots) master

   1 additional replica(s)

S: 4323af513da7a72974ef948b25fc4093054b460f 192.169.0.14:6379

   slots: (0 slots) slave

   replicates bfeb82d5509cba9576942dafda595a56e2e247b8

S: a57402b9a5f7de01a85b47d85fe540efbaa277c7 192.169.0.15:6379

   slots: (0 slots) slave

   replicates 7f4666840dd557177d0b9a2e38f37980e2399aae

M: 5a6bafeef212cab67598403a006cb12e033ed2a3 192.169.0.12:6379

   slots:[5461-10922] (5462 slots) master

   1 additional replica(s)

S: 7c25854b274173ef77847143250ba12d76918f40 192.169.0.16:6379

   slots: (0 slots) slave

   replicates 5a6bafeef212cab67598403a006cb12e033ed2a3

#进入某个redis

docker exec -it redis-1 /bin/sh

#登入到集群

redis-cli -c

#查看信息

cluster info

#查看nodes

cluster nodes

#测试：

set myname zxd 

#然后get myname 失败，重启

/data # redis-cli -c

127.0.0.1:6379> get myname

-> Redirected to slot [12807] located at 192.169.0.14:6379

#"zxd" 从机4获取到

#再查看

192.169.0.14:6379> cluster nodes

7f4666840dd557177d0b9a2e38f37980e2399aae 192.169.0.11:6379@16379 master - 0 1599807281431 1 connected 0-5460

5a6bafeef212cab67598403a006cb12e033ed2a3 192.169.0.12:6379@16379 master - 0 1599807281532 2 connected 5461-10922

a57402b9a5f7de01a85b47d85fe540efbaa277c7 192.169.0.15:6379@16379 slave 7f4666840dd557177d0b9a2e38f37980e2399aae 0 1599807281000 5 connected

bfeb82d5509cba9576942dafda595a56e2e247b8 192.169.0.13:6379@16379 master,fail - 1599807044291 1599807042568 3 connected

7c25854b274173ef77847143250ba12d76918f40 192.169.0.16:6379@16379 slave 5a6bafeef212cab67598403a006cb12e033ed2a3 0 1599807281935 6 connected

4323af513da7a72974ef948b25fc4093054b460f 192.169.0.14:6379@16379 myself,master - 0 1599807280000 8 connected 10923-16383

#3fail 4变成了master
```



## docker发布springboot项目

```shell
#编写Dockerfile

FROM java:8

COPY *.jar /app.jar

CMD ["--server.port=8080"]

EXPOSE 8080

ENTRYPOINT ["java","-jar","/app.jar"]

#然后将package的jar和Dockerfile拷贝到linux中，

#构建： 
docker build -t springboot-test01 .

#运行：
docker run -p 8083:8080 --name springboot-test-app01 springboot-test01

#访问http://192.168.66.100:8083/ppq ok

#问题：只有linux可以访问，外网主机都不行？没有读取到application.xml的端口，而是默认启用了8080
```

## Docker compose

### 安装与授权

```shell
#下载
[root@zxd997 bin]# curl -L https://get.daocloud.io/docker/compose/releases/download/1.25.5/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   423  100   423    0     0    372      0  0:00:01  0:00:01 --:--:--   372
100 16.7M  100 16.7M    0     0   9.8M      0  0:00:01  0:00:01 --:--:-- 30.8M
#授权
[root@zxd997 bin]# sudo chmod +x /usr/local/bin/docker-compose
[root@zxd997 bin]# docker-compose --version
docker-compose version 1.25.5, build 8a1c60f6
```

### 体验

```shell
$ mkdir compose-test
$ cd compose-test
$ vim app.py
$ vim requirements.txt
$ vim Dockerfile
$ vim docker-compose.yml
```

app.py

```python
import time
import redis
from flask import Flask

app = Flask(__name__)
cache = redis.Redis(host='redis', port=6379)


def get_hit_count():
    retries = 5
    while True:
        try:
            return cache.incr('hits')
        except redis.exceptions.ConnectionError as exc:
            if retries == 0:
                raise exc
            retries -= 1
            time.sleep(0.5)


@app.route('/')
def hello():
    count = get_hit_count()
    return 'Hello World! I have been seen {} times.\n'.format(count)

if __name__ == "__main__":
    app.run(host="0.0.0.0",debug=True)
```

requirements.txt

```tex
flask
redis
```

Dockerfile

```tex
FROM python:3.7-alpine
ADD . /code
WORKDIR /code
RUN pip install -r requirements.txt
CMD ["python", "app.py"]
```

docker-compose.yml

```yaml
version: '3'
services:
  web:
    build: .
    ports:
      - "5000:5000"
  redis:
    image: "redis:alpine"
```

docker-compose up 启动

docker-compose down停止

### 编写一个博客

```sh
$ mkdir /home/my-wordpress
$ cd  my-wordpress
$ vim docker-compose.yml
```

```yaml
version: '3.3'

services:
   db:
     image: mysql:5.7
     #可以加个端口映射能从外部访问数据库
     #ports:
       #- "3306:3306"
     volumes:
       - db_data:/var/lib/mysql
     restart: always
     environment:
       MYSQL_ROOT_PASSWORD: somewordpress
       MYSQL_DATABASE: wordpress
       MYSQL_USER: wordpress
       MYSQL_PASSWORD: wordpress

   wordpress:
     depends_on:
       - db
     image: wordpress:latest
     ports:
       - "8000:80"
     restart: always
     environment:
       WORDPRESS_DB_HOST: db:3306
       WORDPRESS_DB_USER: wordpress
       WORDPRESS_DB_PASSWORD: wordpress
       WORDPRESS_DB_NAME: wordpress
volumes:
    db_data: {}
```

启动就ok~

### docker-compose启动微服务项目

创建一个springboot项目，使用web，redis

```shell
$ mkdir my-test-springboot
$ cd my-test-springboot
#拷贝三个文件
[root@zxd997 my-test-springboot]# ls
demo-0.0.1-SNAPSHOT.jar  docker-compose.yml  Dockerfile
```

```java
@RestController
public class TestController {
    @Autowired
    private StringRedisTemplate stringRedisTemplate;
    @GetMapping("/get")
    public String get(){
        Long count = stringRedisTemplate.opsForValue().increment("count", 1);
        return "hello,nihao~,no."+count;
    }
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.3.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.example</groupId>
    <artifactId>demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>demo</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>

```



```properties
#application.properties
server.port=8080
spring.redis.host=redis

#Dockerfile
FROM java:8
COPY *.jar /app.jar
CMD ["--server.port=8080"]
EXPOSE 8080
ENTRYPOINT ["java","-jar","/app.jar"]
```

```yaml
#docker-compose.yml
version: '3.8'
services: 
  kuangapp:
    build: .
    image: kuangapp
    depends_on:
      - redis
    ports:
      - "8080:8080"
  redis:
    image: "library/redis:alpine"
```

docker-compose up 启动

如果项目重新部署打包

```shell
docker-compose up --build
#-d 后台启动
docker-compose up --build -d
```



## Docker swarm

集群

