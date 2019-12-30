# Nginx

## https

[在Nginx服务器安装证书](https://help.aliyun.com/document_detail/98728.html)

- 将证书下载至 `nginx/cert`

```shell script
$ cp domain_name.pem nginx/cert
$ cp domain_name.key nginx/cert
```

- 编辑 `nginx/conf.d/default.conf` 替换其中的 `domain_name`

```
server {
    listen 80;
    server_name domain_name; # 将domain_name修改为证书绑定的域名
    rewrite ^(.*)$ https://$host$1 permanent; # 设置HTTP请求自动跳转HTTPS

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
}

# HTTPS server
server {
    listen 443 ssl;
    server_name domain_name; # 将domain_name修改为证书绑定的域名
    ssl_certificate cert/domain_name.pem; # 将domain_name.pem替换成证书的文件名
    ssl_certificate_key cert/domain_name.key; # 将domain_name.key替换成证书的密钥文件名
    ssl_session_timeout 5m;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
}
```

本机测试

```shell script
$ sudo vi /etc/hosts
127.0.X.1	domain_name;
$ sudo /etc/init.d/networking restart
$ sudo docker restart nginx
```

## docker

[Docker Hub](https://hub.docker.com/_/nginx)

### docker

```shell script
$ sudo docker pull nginx
$ sudo docker run -d --name nginx -p 80:80 nginx
$ sudo docker exec -it nginx bash
$ sudo docker cp nginx:/etc/nginx/nginx.conf ~/appdata/nginx
$ sudo docker cp nginx:/etc/nginx/conf.d ~/appdata/nginx/conf.d
$ sudo docker cp nginx:/usr/share/nginx/html ~/appdata/nginx/html
$ sudo docker stop nginx
$ sudo docker rm nginx
```

### docker-compose

- stack.yml

```yaml
version: '3.1'

services:
  nginx:
    image: nginx
    restart: always
    hostname: nginx
    container_name: nginx
    ports:
      - 80:80
      - 443:443
    volumes:
      - ${HOME}/appdata/nginx/nginx.conf:/etc/nginx/nginx.conf
      - ${HOME}/appdata/nginx/conf.d:/etc/nginx/conf.d
      - ${HOME}/appdata/nginx/cert:/etc/nginx/cert
      - ${HOME}/appdata/nginx/html:/usr/share/nginx/html
      - ${HOME}/appdata/nginx/log:/var/log/nginx
    environment:
      - TZ=Asia/Shanghai
```

- 运行

```shell script
$ sudo docker pull nginx
$ sudo docker-compose -f stack.yml up -d
```