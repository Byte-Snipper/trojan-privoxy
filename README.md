# trojan-privoxy

trojan client with privoxy for http proxy

## How to use

```bash
# https://stackoverflow.com/questions/36075525/how-do-i-run-a-docker-instance-from-a-dockerfile
# create image from Dockerfile, name is trojan_privoxy
docker build --tag trojan_privoxy .
docker image ls

docker run \
    --name my_proxy \
    -e REMOTE_ADDR="your host" \
    -e REMOTE_PORT="server port" \
    -e PASSWORD="your password" \
    -p TROJAN_PORT:1086 \
    -p PRIVOXY_PORT:1087 \
    trojan_privoxy:latest
```

TROJAN_PORT and PRIVOXY_PORT are proxy ports on HOST.

## use proxy

```bash
export http_proxy=http://127.0.0.1:PRIVOXY_PORT
export https_proxy=http://127.0.0.1:PRIVOXY_PORT
```

## with proxychains
```bash
apt install proxychains

# edit /etc/proxychains.conf
sed -i '/^socks4/d' /etc/proxychains.conf
echo "socks5 127.0.0.1 TROJAN_PORT" >> /etc/proxychains.conf
proxychains curl -4 ip.sb
```

## chain proxy B from A
B(172.50.2.33)为上述方式搭建的代理，C(10.157.21.62)服务器无法直连B，可以通过A(192.168.1.155)进行转发：使用 GOST 进行二级代理
```bash
# 1. 下载 (针对 x86_64 架构，如果是 ARM 请替换文件名)
curl -L https://github.com/ginuerzh/gost/releases/download/v2.11.5/gost-linux-amd64-2.11.5.gz -o gost.gz

# 2. 解压
gunzip gost.gz

# 3. 赋予执行权限并移动到系统路径
chmod +x gost
mv gost /usr/bin/gost

# 4. 测试
gost -V

# 在服务器 B 上开启 21087 端口作为入口，并将所有流量通过 A 转发
gost -L=:21087 -F=http://172.50.2.33:21087

# 在C上
export http_proxy=http://192.168.1.155:21087
```
