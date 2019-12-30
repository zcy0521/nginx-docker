# Nginx

## Usage

```shell script
# 安装docker-compose
sudo curl -L "https://github.com/docker/compose/releases/download/1.24.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# 下载证书 替换default.conf中domain_name
git clone https://github.com/zcy0521/nginx-docker.git
cd nginx-docker
mkdir cert
cp domain_name.pem domain_name.key cert/
vi conf.d/default.conf

# 运行 nginx
sudo docker pull nginx
sudo docker-compose -f stack.yml up -d
```

## Docker

[Docker Hub](https://hub.docker.com/_/nginx)

```shell script
$ sudo docker pull nginx
$ sudo docker run -d --name nginx -p 80:80 nginx
$ sudo docker exec -it nginx bash
$ sudo docker stop nginx
$ sudo docker rm nginx
```

## Https

[在Nginx服务器安装证书](https://help.aliyun.com/document_detail/98728.html)

- 将证书下载至 `nginx/cert`

```shell script
mkdir cert
cp domain_name.pem domain_name.key cert/
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

- 本机测试

```shell script
sudo vi /etc/hosts
127.0.X.1	domain_name;
sudo /etc/init.d/networking restart
sudo docker restart nginx
```
