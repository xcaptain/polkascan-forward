# nginx转发substrate ws接口

substrate程序启动后，对外的接口默认是监听的 `127.0.0.1:9944`，没有直接的配置能将之修改为`0.0.0.0:9944`，本项目的目的就是通过nginx将网络中转一下，从而提供外网可访问的接口地址。

## 生成自签发证书

```shell
mkdir -p ssl/certs
mkdir -p ssl/private

openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout ./ssl/private/nginx
-selfsigned.key -out ./ssl/certs/nginx-selfsigned.crt

openssl dhparam -out ./ssl/certs/dhparam.pem 2048
```

此时 `ssl/` 目录结构如下：
```markdown
ssl
├── certs
│   ├── dhparam.pem
│   └── nginx-selfsigned.crt
└── private
    └── nginx-selfsigned.key
```

## 启动nginx容器

```shell
docker run -d -p 9945:9945 -p 9955:9955 -v "$(pwd)/conf.d:/etc/nginx/conf.d" -v "$(pwd)/ssl:/etc/nginx/ssl" --name polkascan-forward --network=host nginx
```

将本项目下的nginx配置文件和自签发证书都挂载到容器里，然后对外暴露9945这个端口。注意，因为容器和宿主机不在一个网络内，因此如果要让容器能访问到宿主机的127.0.0.1，需要加上`--network=host` 这个选项

## 排查问题

常见问题包括：
1. 因为是自签发证书，浏览器并不信任，需要手动在浏览器访问这个地址，设置为信任
2. 如果是返回502，那要检查下是不是容器内没法访问到宿主机的127.0.0.1:9944，可以用telnet这条命令测试
