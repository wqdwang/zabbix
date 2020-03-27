由于前两天搭建了一个博客网站，想要监控服务器状态，闲着没有事情做，这里打算使用docker搭建zabbix service 进行监控，也顺便学习下docker。

1. ## 搭建docker ##


1.2 卸载旧版本
较旧的 Docker 版本称为 docker 或 docker-engine 。如果已安装这些程序，请卸载它们以及相关的依赖项。
    
    sudo yum remove docker \
      docker-client \
      docker-client-latest \
      docker-common \
      docker-latest \
      docker-latest-logrotate \
      docker-logrotate \
      docker-engine

设置仓库，安装所需的软件包
1.3: 添加docker的YUM源
    `yum install -y yum-utils device-mapper-persistent-data lvm2`

1.4: 这里使用阿里云源进行安装
    yum-config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
     
    yum install docker-ce -y

查看docker 是否安装成功,如下表示安装成功。

1.5 更换源
顺便把 docker 源也换一下吧，免得一会还的重启

    [root@zabbix~]# vim /etc/docker/daemon.json 
    {
     "registry-mirrors": ["https://mirror.ccs.tencentyun.com"]
    }
    

2. 搭建zabbix之前我们要知道，使用docker搭建zabbix的几个组件：



数据库:MySQL 或是 PostgreSQL
Zabbix Java gateway	Java:管理扩展，可以不添加
Zabbix server-zabbix:服务端
zabbix-web与MySQL服务器实例和 Zabbix server 实例关联
zabbix-agentzabbix:客户端

docker run --name mysql-server -t \
      -v /root/zabbix/data:/var/lib/mysql \
      -e MYSQL_DATABASE="zabbix" \
      -e MYSQL_USER="zabbix" \
      -e MYSQL_PASSWORD="Dong.0326" \
      -e MYSQL_ROOT_PASSWORD="Dong.0326" \
      -d mysql:5.7\
      --character-set-server=utf8 --collation-server=utf8_bin
 
docker run --name zabbix-java-gateway -t \
      -d zabbix/zabbix-java-gateway:latest
 
 
docker run --name zabbix-server-mysql -t \
      -e DB_SERVER_HOST="mysql-server" \
      -e MYSQL_DATABASE="zabbix" \
      -e MYSQL_USER="zabbix" \
      -e MYSQL_PASSWORD="Dong.0326" \
      -e MYSQL_ROOT_PASSWORD="Dong.0326" \
      -e ZBX_JAVAGATEWAY="zabbix-java-gateway" \
      --link mysql-server:mysql \
      --link zabbix-java-gateway:zabbix-java-gateway \
      -p 10051:10051 \
      -d zabbix/zabbix-web-nginx-mysql:centos-4.0.4  
 
 
docker run --name zabbix-web-nginx-mysql -t \
      -e DB_SERVER_HOST="mysql-server" \
      -e MYSQL_DATABASE="zabbix" \
      -e MYSQL_USER="zabbix" \
      -e MYSQL_PASSWORD="Dong.0326" \
      -e MYSQL_ROOT_PASSWORD="Dong.0326" \
      --link mysql-server:mysql \
      --link zabbix-server-mysql:zabbix-server \
      -p 80:80 \
      -d zabbix/zabbix-web-nginx-mysql:latest
 
 
 
docker run --name zabbix-agent \
            -e ZBX_HOSTNAME="111.230.25.248" \
            -e ZBX_SERVER_HOST="49.232.150.5" \
            -e ZBX_METADATA="zabbix"
            -p 10050:10050 \
            --privileged
            -d zabbix/zabbix-agent:latest
