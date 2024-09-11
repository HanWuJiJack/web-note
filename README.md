创建用户定义的网桥网络my-net。
docker network  create --driver bridge my-net

运行容器
docker run -d --name nps1 -p 8180:8080 -p 8124:8014 --network my-net 52eb

mysql57 安装
-v /mydocker/mysql/conf:/etc/mysql/conf.d ：将主机/mydocker/mysql录下的conf/my.cnf 挂载到容器的 /etc/mysql/conf.d
-v /mydocker/mysql/logs:/var/log/mysql：将主机/mydocker/mysql目录下的 logs 目录挂载到容器的 /logs。
-v /mydocker/mysql/data:/var/lib/mysql ：将主机/mydocker/mysql目录下的data目录挂载到容器的 /var/lib/mysql


docker run --name mysql80 -v ~/dev/env/docker/mysql80/conf:/etc/mysql/conf.d -v ~/dev/env/docker/mysql80/logs:/var/log/mysql -v ~/dev/env/docker/mysql80/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -p 3305:3306 -d 23b0


docker pull mysql:5.7

docker run --name mysql57-Slave2 -e MYSQL_ROOT_PASSWORD=123456 -p 63306:3306 -d  c209

<!-- 导出数据 -->
mysql -uroot -p123456 </home/all123.sql


nps 通过中间服务器 访问客户端
搭建NPS服务端
docker pull ffdfgdfg/nps
docker run -d --name nps2 -p 8180:8080 -p 8124:8014 --restart=always --network my-net -v /Users/shun/Documents/软件收藏/nps/darwin_amd64_server/conf:/conf ffdfgdfg/nps
# 查看日志
docker logs nps


客户端安装使用
docker pull ffdfgdfg/npc
docker run -d --name=npc3 --restart=always --network my-net ffdfgdfg/npc -server=172.18.0.2:8024 -vkey=zqsqvyb54oj27oni
# 查看日志
docker logs npc


配置前端项目

docker run -it --name web -p 8480:80 -p 8422:22 --restart=always --network my-net -v /Users/shun/Documents/project/docker/web:/home/hsueh/web f478 /bin/bash

service nginx start

docker container start xxxx
docker container stop xxxx

docker attach web
docker container ls

nginx 镜像 安装nvm

apt update -y

apt install git openssh-server openssl vim -y


安装 NVM
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.37.2/install.sh | bash

ssh 安装
https://www.jianshu.com/p/51251b30349c

<!-- 导出 -->
docker export 0c4d > centos7.tar
docker export web > centos7.tar
<!-- 导入 -->
cat centos7.tar | docker import - hsueh/centos7:v1.0

cat centos7.tar | docker import - hsueh/web:v1.0
<!-- 运行 -->
docker run -d --name centos7 -p 8180:8080 -p 8122:22 --network my-net 164b /usr/sbin/sshd docker container ls-D


## rabbitmq
docker search rabbitmq:management
docker pull rabbitmq:management
docker images
docker run --name myrabbitmq -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=admin123 -p 15672:15672 -p 5672:5672 -d rabbitmq:management

docker run -d -v D:/docker/rabbitmq/lib:/var/lib/rabbitmq -p 5672:5672 -p 15672:15672 --name rabbitmq rabbitmq:management

http://localhost:15672/ guest/guest


## redis
docker run -d --name redis-stack-server -p 6379:6379 -v ~/dev/env/docker/redis/redis.conf:/etc/redis/redis.conf  -v ~/dev/env/docker/redis/data:/data redis/redis-stack-server:latest


## mongo
docker pull mongo:latest

docker images

docker run -d --name mongodb -p 27017:27017 -v ~/dev/env/docker/mongodb:/data/db -e MONGO_INITDB_ROOT_USERNAME=admin -e MONGO_INITDB_ROOT_PASSWORD=admin123  mongo


docker run -d --name mongodb -p 27017:27017 -v Users/shun/docker/mongodb:/data/db -e MONGO_INITDB_ROOT_USERNAME=admin -e MONGO_INITDB_ROOT_PASSWORD=admin123 --privileged=true --restart always mongo
<!-- https://www.jianshu.com/p/d5daf8b88bb4 -->


## ffmpeg

docker run -itd --name app_ffmpeg -p 8066:8080 -v /Users/shun/docker/ffmpeg/:/mnt/app/ --entrypoint='bash' jrottenberg/ffmpeg



### onlyoffice
docker load -i oo73340.tar

docker run -d -p 18089:80 -p 18090:22 --name=onlyoffice --restart=always 镜像ID
docker run -d -p 18089:80 --name=onlyoffice --restart=always b89b
ubuntu 22.04开机启动
vim /etc/rc.local
/usr/sbin/sshd -D &

压缩文件的指令：tar -zcvf targetFiles.tar.gz targetFiles
解压文件的指令：tar -zxvf targetFiles.tar.gz（默认解压到文件所在路径）
tar -zcvf app.js.gz app.js
tar -zcvf app.css.gz app.css
查看端口
netstat -ap
netstat -ap | grep 8080



docker export bf7f > onlyoffice.tar  

## mysql主从复制功能搭建  
<!-- https://segmentfault.com/a/1190000023775512 -->
查看ip  
hostname -I  



master:172.17.0.2
s1:172.17.0.3
s2:172.17.0.4


master 主机操作
<!-- GRANT REPLICATION SLAVE ON *.* TO 'slave_user'@'slave_host' IDENTIFIED BY 'slave_password'; -->
slave_user:master用户名
slave_password:master密码
slave_host：从机ip

```
GRANT REPLICATION SLAVE ON *.* to 'root'@'172.17.0.3' identified by '123456';
GRANT REPLICATION SLAVE ON *.* to 'root'@'172.17.0.4' identified by '123456';
//刷新系统权限表的配置
FLUSH PRIVILEGES;
```

接下来在找到mysql的配置文件/etc/my.cnf，增加以下配置：

```
# 开启binlog
log-bin=mysql-bin
server-id=2
# 需要同步的数据库，如果不配置则同步全部数据库
binlog-do-db=test_db
# binlog日志保留的天数，清除超过10天的日志
# 防止日志文件过大，导致磁盘空间不足
expire-logs-days=10 
```

一定要重启后再操作

mysql -u root -p
show master status\G;

可以通过命令行show master status\G;查看当前binlog日志的信息(获取File、Position)：

File: mysql-bin.000002
         Position: 154
     Binlog_Do_DB: test_db
 Binlog_Ignore_DB: 
Executed_Gtid_Set: 
1 row in set (0.00 sec)


Slave配置
找到/etc/my.cnf配置文件，增加以下配置：

```
server-id=106  
```
一定要重启后再操作  
```
mysql -u root -p  
```

进入到mysql后，再输入以下命令：  

```
CHANGE MASTER TO 
MASTER_HOST='172.17.0.2',
MASTER_USER='root',
MASTER_PASSWORD='123456',
MASTER_LOG_FILE='mysql-bin.000002',
MASTER_LOG_POS=154,
master_port=3306;
```

启动slave服务  
```
start slave;
```

查看slave服务状态 
```
show slave status\G;
```

参数解释

CHANGE MASTER TO 
MASTER_HOST='172.17.0.2',//主机IP
MASTER_USER='root',//之前创建的用户账号
MASTER_PASSWORD='123456',//之前创建的用户密码
MASTER_LOG_FILE='mysql-bin.000002',//master主机的binlog日志名称
MASTER_LOG_POS=154,//binlog日志偏移量
master_port=3306;//端口


## 容器自启
docker update --restart=always docker_id
docker update --restart=no docker_id


## ubuntu
docker run --name ubuntu1 -ti -d -p 8067:80 -p 8068:22  ba6a

curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/master/install.sh | bash
export NVM_DIR="$([ -z "${XDG_CONFIG_HOME-}" ] && printf %s "${HOME}/.nvm" || printf %s "${XDG_CONFIG_HOME}/nvm")"
[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"


## imageAi
docker run --name modelscope -p 7860:7860 -e "types=removebg,changebg,repair-photo,human-cartoon,video-cartoon,ocr,cntext2image" -v /Users/shun/docker/imageai:/mnt/workspace/.cache -d sakoo123/modelscope:1.0

docker run --name  modelscope2  -p 7861:7860 -v /Users/shun/docker/imageai:/mnt/workspace/.cache     -d   sakoo123/modelscope:1.0


## 开源的离线OCR
https://github.com/alisen39/TrWebOCR

### 从 dockerhub pull
docker pull mmmz/trwebocr:latest

### 运行镜像
docker run -d --rm -p 8089:8089 --name trwebocr mmmz/trwebocr:latest 


### 运行consul镜像
docker pull consul:1.15.4

docker run -d --name=consul \
-p 8500:8500 \
-p 8300:8300 \
-p 8301:8301 \
-p 8302:8302 \
-p 8360:8360/udp \
consul:1.15.4 agent -dev -client=0.0.0.0
<!-- 设置自动重启 -->
docker container update --restart==always consul


docker pull vaalacat/frp-panel 

docker run -d -p 9000:9000 -p 9001:9001 -p 9002:7000 -p 20000-20050:20000-20050 \
  --restart=unless-stopped \
  -v /Users/hsueh/dev/env/docker/frp-panel:/data \
  -e APP_GLOBAL_SECRET=123456 \
  -e MASTER_RPC_HOST=192.168.10.135 \
  vaalacat/frp-panel

docker run --detach \
  --publish 1270:1270 --publish 1271:1271 \
  -p 2270-3270:1270-2270 \
  --name FastTunnel \
  --restart always \
  --volume /Users/hsueh/dev/env/docker/FastTunnel/config:/app/config \
  --volume /Users/hsueh/dev/env/docker/FastTunnel/Logs:/app/Logs \
  springhgui/fasttunnel:latest