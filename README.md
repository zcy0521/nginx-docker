# Nginx Docker

## Usage

- 启动

```shell script
git clone https://github.com/zcy0521/nginx-docker.git
cd nginx-docker
sudo docker-compose up -d
sudo docker-compose ps
```

- 删除

```shell script
sudo docker-compose down
```

## Docker

### Install

- [Debian](https://docs.docker.com/engine/install/debian)

```shell script
sudo apt-get update
sudo apt-get install apt-transport-https ca-certificates curl gnupg-agent software-properties-common
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
sudo apt-key fingerprint 0EBFCD88
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/debian $(lsb_release -cs) stable"
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
sudo docker run hello-world
```

- [Ubuntu](https://docs.docker.com/engine/install/ubuntu/)

```shell script
sudo apt-get update
sudo apt-get install apt-transport-https ca-certificates curl gnupg-agent software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo apt-key fingerprint 0EBFCD88
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
sudo docker run hello-world
```

### Docker Compose

- [Linux](https://docs.docker.com/compose/install/#install-compose-on-linux-systems)

```shell script
sudo curl -L "https://github.com/docker/compose/releases/download/1.26.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

### Docker run Nginx

[Docker Hub](https://hub.docker.com/_/nginx)

```shell script
sudo docker run -d --name nginx -p 80:80 nginx
```

## Nginx

### Https

[Configuring HTTPS servers](http://nginx.org/en/docs/http/configuring_https_servers.html)

[在Nginx服务器安装阿里云SSL证书](https://help.aliyun.com/document_detail/98728.html)

- 将证书文件上传至服务器

```shell script
cp <DOMAIN_NAME>.pem cert/
cp <DOMAIN_NAME>.key cert/
```

- 新建配置文件`conf.d/`

```shell script
nano conf.d/<DOMAIN_NAME>.conf

server {
    listen       80;
    server_name  <DOMAIN_NAME>;
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
    server_name <DOMAIN_NAME>;

    ssl_certificate cert/<DOMAIN_NAME>.pem;
    ssl_certificate_key cert/<DOMAIN_NAME>.key;
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

- 新建配置文件

```shell script
nano conf.d/<DOMAIN_NAME>.conf

upstream <APP_NAME> {
    server <TOMCAT_SERVER>:<TOMCAT_PORT>;
    server <TOMCAT_SERVER>:<TOMCAT_PORT>;
    ...
}

server {
    listen 443 ssl;
    server_name <DOMAIN_NAME>;

    ssl_certificate cert/<DOMAIN_NAME>.pem;
    ssl_certificate_key cert/<DOMAIN_NAME>.key;
    ssl_session_timeout 5m;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;

    location / {
        proxy_pass http://<APP_NAME>;
    }
}
```

- 重启nginx

```shell script
sudo docker restart nginx
```
