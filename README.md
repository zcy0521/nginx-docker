# Nginx

## Usage

```shell script
# 安装docker-compose
sudo curl -L "https://github.com/docker/compose/releases/download/1.24.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

git clone https://github.com/zcy0521/nginx-docker.git
cd nginx-docker

# 下载证书 替换ssl.conf中domain_name
mkdir cert
cp domain_name.pem domain_name.key cert/
vi conf.d/ssl.conf

# 替换tomcat.conf中[webappname]
vi conf.d/tomcat.conf

# 运行 nginx
sudo docker pull nginx
sudo docker-compose -f stack.yml up -d

# 修改 hosts
sudo vi /etc/hosts
127.0.X.1	domain_name
sudo /etc/init.d/networking restart

# 访问
https://domain_name
```

## Docker

[Docker Hub](https://hub.docker.com/_/nginx)

```shell script
sudo docker pull nginx
sudo docker run -d --name nginx -p 80:80 nginx
sudo docker exec -it nginx bash
sudo docker stop nginx
sudo docker rm nginx
```

## Nginx

### Https

- 将证书下载至 `nginx/cert`

```shell script
mkdir cert
cp domain_name.pem domain_name.key cert/
```

- 编辑 `conf.d/ssl.conf` 替换其中的 `domain_name`

[Configuring HTTPS servers](http://nginx.org/en/docs/http/configuring_https_servers.html)

[在Nginx服务器安装阿里云SSL证书](https://help.aliyun.com/document_detail/98728.html)

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

### upstream

- 连接tomcat

[Manage a user-defined bridge](https://docs.docker.com/network/bridge/#manage-a-user-defined-bridge)

[Specify custom networks](https://docs.docker.com/compose/networking/#specify-custom-networks)

Networks can also be given a [custom name](https://docs.docker.com/compose/compose-file/#name-1) (since version 3.5):

```yaml
version: "3.7"
networks:
  my-app-net:
    name: my-app-net
```

- 编辑 `conf.d/tomcat.conf` 替换其中的 `[webappname]`

[upstream](http://nginx.org/en/docs/http/ngx_http_upstream_module.html)

```
upstream [webappname] {
    server localhost:8081;
    server localhost:8082;
}

server {
    listen 80;
    server_name [webappname];

    location / {
        proxy_pass http://[webappname];
    }
}
```
