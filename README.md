# Nginx

## Usage

- 运行 nginx

```shell script
git clone https://github.com/zcy0521/nginx-docker.git
cd nginx-docker
sudo docker pull nginx
sudo docker-compose up -d
```

- 配置 `[DOMAIN_NAME].conf`

```shell script
touch conf.d/[DOMAIN_NAME].conf
vi conf.d/[DOMAIN_NAME].conf
sudo docker restart nginx
```

## Docker

- [Get Docker Engine - Community for Debian](https://docs.docker.com/install/linux/docker-ce/debian/)

```shell script
sudo apt-get update
sudo apt-get install apt-transport-https ca-certificates curl gnupg2 software-properties-common
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
sudo apt-key fingerprint 0EBFCD88
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/debian $(lsb_release -cs) stable"
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
sudo docker run hello-world
```
  
- [Install Docker Compose](https://docs.docker.com/compose/install/)

```shell script
sudo curl -L "https://github.com/docker/compose/releases/download/1.25.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

## Nginx

- [Docker Hub](https://hub.docker.com/_/nginx)

```shell script
sudo docker pull nginx
sudo docker run -d --name nginx -p 80:80 nginx
sudo docker exec -it nginx bash
sudo docker stop nginx
sudo docker rm nginx
```

### Https

- 编辑 `conf.d/[DOMAIN_NAME].conf`

[Configuring HTTPS servers](http://nginx.org/en/docs/http/configuring_https_servers.html)

[在Nginx服务器安装阿里云SSL证书](https://help.aliyun.com/document_detail/98728.html)

```
server {
    listen 80;
    server_name [DOMAIN_NAME]; # 将[DOMAIN_NAME]修改为证书绑定的域名
    rewrite ^(.*)$ https://$host$1 permanent; # 设置HTTP请求自动跳转HTTPS

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
}

server {
    listen 443 ssl;
    server_name [DOMAIN_NAME]; # 将[DOMAIN_NAME]修改为证书绑定的域名
    ssl_certificate cert/[DOMAIN_NAME].pem; # 将[DOMAIN_NAME].pem替换成证书的文件名
    ssl_certificate_key cert/[DOMAIN_NAME].key; # 将[DOMAIN_NAME].key替换成证书的密钥文件名
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

- 将证书下载至 `nginx/cert`

```shell script
cp [DOMAIN_NAME].pem [DOMAIN_NAME].key cert/
```

### upstream

- 编辑 `conf.d/[DOMAIN_NAME].conf`

[upstream](http://nginx.org/en/docs/http/ngx_http_upstream_module.html)

```
upstream [DOMAIN_NAME] {
    server 127.0.0.1:8081;
    server 127.0.0.1:8082;
}

server {
    listen       81;
    server_name  localhost;

    location / {
        proxy_pass http://[DOMAIN_NAME];
    }
}
```
