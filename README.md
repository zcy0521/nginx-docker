# Nginx Docker

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

- nginx

```shell script
sudo docker pull nginx
sudo docker run -d --name nginx -p 80:80 nginx
sudo docker exec -it nginx bash
sudo docker stop nginx
sudo docker rm nginx
```

## Usage

```shell script
git clone https://github.com/zcy0521/nginx-docker.git
cd nginx-docker
sudo docker-compose up -d
```

## Nginx

### Https

[Configuring HTTPS servers](http://nginx.org/en/docs/http/configuring_https_servers.html)

[在Nginx服务器安装阿里云SSL证书](https://help.aliyun.com/document_detail/98728.html)

- 将证书文件上传至服务器

```shell script
cp [DOMAIN_NAME].pem cert/
cp [DOMAIN_NAME].key cert/
```

- 新建配置文件

```shell script
touch conf.d/[DOMAIN_NAME].conf
nano conf.d/[DOMAIN_NAME].conf
server {
    listen       80;
    server_name  [DOMAIN_NAME];
    rewrite ^(.*)$ https://$host$1 permanent;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}

# HTTPS server
server {
    listen 443 ssl;
    server_name [DOMAIN_NAME];
    ssl_certificate cert/[DOMAIN_NAME].pem;
    ssl_certificate_key cert/[DOMAIN_NAME].key;
    ssl_session_timeout 5m;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;

    location / {
            root   /usr/share/nginx/html;
            index  index.html index.htm;
        }
    
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```

- 重启nginx

```shell script
sudo docker restart nginx
```

### upstream

[upstream](http://nginx.org/en/docs/http/ngx_http_upstream_module.html)

- 部署tomcat项目

- 新建配置文件

```shell script
touch conf.d/[DOMAIN_NAME].conf
nano conf.d/[DOMAIN_NAME].conf
upstream [APP_NAME] {
    server 127.0.0.1:[TOMCAT_PORT];
    server 127.0.0.1:[TOMCAT_PORT];
}

server {
    listen       [APP_PORT];
    server_name  localhost;

    location / {
        proxy_pass http://[APP_NAME];
    }
}
```

- 重启nginx

```shell script
sudo docker restart nginx
```
