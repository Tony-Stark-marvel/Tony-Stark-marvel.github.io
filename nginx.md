通过修改conf文件夹下的nginx.conf文件进行配置

```
 server {
        listen       80; 
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            proxy_pass  http://127.0.0.1:8199/hello;		
            index  index.html index.htm;
        }
```

server中listen和name是监听的端口和服务名ip,location是需要代理的服务