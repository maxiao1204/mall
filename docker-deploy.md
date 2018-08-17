#docker环境部署

##docker环境安装
###docker安装
1. 安装yum-utils：
yum install -y yum-utils \
device-mapper-persistent-data \
lvm2
2. 为yum源添加docker仓库位置：
yum-config-manager \
--add-repo \
https://download.docker.com/linux/centos/docker-ce.repo
3. 安装docker:
yum install docker-ce
4. 启动docker:
systemctl start docker
注：常见命令见macro/spring-cloud-demo中的docker.md
###docker compose安装
1. 下载地址：https://github.com/docker/compose/releases
2. 安装地址：/usr/local/bin/docker-compose
3. 设置为可执行：sudo chmod +x /usr/local/bin/docker-compose
4. 测试是否安装成功：docker-compose --version

##mysql安装
###下载镜像文件
docker pull mysql:5.7
###创建实例并启动
docker run -p 3306:3306 --name mysql \
-v /mydata/mysql/log:/var/log/mysql \
-v /mydata/mysql/data:/var/lib/mysql/ \
-v /mydata/mysql/conf:/etc/mysql \
-e MYSQL_ROOT_PASSWORD=123456  \
-d mysql:5.7
> 参数说明
- -p 3306:3306：将容器的3306端口映射到主机的3306端口
- -v /mydata/mysql/conf:/etc/mysql：将配置文件夹挂在到主机
- -v /mydata/mysql/log:/var/log/mysql：将日志文件夹挂载到主机
- -v /mydata/mysql/data:/var/lib/mysql/：将配置文件夹挂载到主机
- -e MYSQL_ROOT_PASSWORD=123456：初始化root用户的密码
###通过容器的mysql命令行工具连接
docker exec -it mysql mysql -uroot -p123456
###设置远程访问
grant all privileges on *.* to 'root' @'%' identified by 'root';
flush privileges;
###进入容器文件系统
docker exec -it mysql /bin/bash

##redis安装
###下载镜像文件
docker pull redis:3.2
###创建实例并启动
docker run -p 6379:6379 --name redis -v /mydata/redis/data:/data -d redis:3.2 redis-server --appendonly yes
###使用redis镜像执行redis-cli命令连接
docker exec -it redis redis-cli

##nginx安装
###下载镜像文件
docker pull nginx:1.10
###创建实例并启动
docker run -p 80:80 --name nginx \
-v /mydata/nginx/html:/usr/share/nginx/html \
-v /mydata/nginx/logs:/var/log/nginx  \
-v /mydata/nginx/conf:/etc/nginx \
-d nginx:1.10
###修改nginx配置
1. 将容器内的配置文件拷贝到当前目录：docker container cp nginx:/etc/nginx .
2. 修改文件名称：mv nginx conf
3. 终止容器：docker stop nginx
4. 执行以下命令：
docker run -p 80:80 --name nginx \
-v /mydata/nginx/html:/usr/share/nginx/html \
-v /mydata/nginx/logs:/var/log/nginx  \
-v /mydata/nginx/conf:/etc/nginx \
-d nginx:1.10

##rabbitmq安装
###下载镜像文件
docker pull rabbitmq:management
###创建实例并启动
docker run -d --name rabbitmq --publish 5671:5671 \
 --publish 5672:5672 --publish 4369:4369 --publish 25672:25672 --publish 15671:15671 --publish 15672:15672 \
rabbitmq:management

##elasticsearch安装
###下载镜像文件
docker pull elasticsearch:2.4
###创建实例并运行
docker run -p 9200:9200 -p 9300:9300 --name elasticsearch -d elasticsearch:2.4
###测试
访问会返回版本信息：http://192.168.1.66:9200/
###安装目录位置
/usr/share/elasticsearch
###安装head插件
1. 进入docker内部bash:docker exec -it elasticsearch /bin/bash
2. 安装插件：plugin install mobz/elasticsearch-head
3. 测试：http://192.168.1.66:9200/_plugin/head/
###安装中文分词器IKAnalyzer
1. 下载中文分词器：https://github.com/medcl/elasticsearch-analysis-ik/releases?after=v5.6.4
2. 上传后拷贝到容器中：docker container cp elasticsearch-analysis-ik-1.10.6.tar.gz elasticsearch:/usr/share/elasticsearch/plugins
3. 进行解压操作：tar -xvf elasticsearch-analysis-ik-1.10.6.tar.gz
4. 重新启动容器：docker restart elasticsearch
5. 测试：
POST:http://192.168.1.66:9200/_analyze
JSON:{"analyzer":"ik","text":"联想是全球最大的笔记本厂商"}

##mongodb安装
###下载镜像文件
docker pull mongo:3.2
###创建实例并运行
docker run -p 27017:27017 --name mongo -v $PWD/db:/data/db -d mongo:3.2
###使用mongo命令进入容器
docker exec -it mongo mongo