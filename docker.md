### 拉取centos镜像

```
docker pull centos:centos7
```

查看镜像文件

![image-20220725095808892](C:\Users\zhengao\AppData\Roaming\Typora\typora-user-images\image-20220725095808892.png)

#### docker启动mysql服务

```
docker run -itd --name mysql-test -p 3308:3306 -e MYSQL_ROOT_PASSWORD=123456 mysql
```

#### docker 启动redis服务

```
docker run -itd --name redis-test -p 6380:6379 redis
```

### 在centos镜像中安装mysql和redis服务

> 开启容器

```
docker run -d -name centos7 --privileged=true centos:7 /usr/sbin/init
```

> 进入容器

```
docker exec -it centos7 /bin/bash
```

安装MySQL服务

[centos安装mysql]: https://www.cnblogs.com/leecy/p/16328065.html

安装redis服务

[安装redis]: http://t.zoukankan.com/RyanJin-p-9777565.html

#### 打包

> 打包之前先把容器停掉

之后利用docker commit 命令

```
docker commit -a "tony" -m "centos-server" 4a5af4af8b20 centos-server:v1
```

> 镜像打包

```
docker save -o 要打包名字.tar 镜像名字:版本号
docker save -o centos-server.tar centos-server:v1
```

> 打包解压成镜像

```
docker load <  已打包的镜像名字.tar
docker load < centos-server.tar
```

#### dockerfile 构建es命令

```
FROM docker.elastic.co/elasticsearch/elasticsearch:6.2.3
RUN elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v6.2.3/elasticsearch-analysis-ik-6.2.3.zip 
RUN elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-pinyin/releases/download/v6.2.3/elasticsearch-analysis-pinyin-6.2.3.zip
CMD ["cron", "-f"]
```

> docker build 命令执行

#### 启动镜像

```
docker run --restart=always -d -e ES_JAVA_POTS="-Xms256m -Xmx256m" -e "discovery.type=single-node" -p 9200:9200 -p 9300:9300 --name ES3 502bab3f5e7d
```

