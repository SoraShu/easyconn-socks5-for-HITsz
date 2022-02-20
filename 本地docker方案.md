# 本地docker方案

## 在wsl中安装docker

推荐使用wsl2版本，推荐使用ubuntu发行版。wsl是没有systemd的，但ubuntu的wsl有一个魔改的service，在使用docker上方便不少。

使用docker-cli，并不想装docker desktop。使用wsl2，因为我在用wsl2。

以下教程默认发行版为ubuntu for wsl2，并且安装了docker-ce以及docker-compose，docker-ce安装可以参考[ustc镜像站-Docker CE 源使用帮助](https://mirrors.ustc.edu.cn/help/docker-ce.html)。

推荐更换dockerhub源，如阿里云提供的加速服务。

可以像在ubuntu里面使用一样使用`service docker start`等命令启动或者重启docker服务。

## 使用docker-compose部署

文件结构
```
easyconnect
├── .easyconn
└── docker-compose.yml
```
`.easyconn`文件置空即可，为easyconnect的配置文件。

docker-compose配置文件如下：
```yml
# docker-compose.yml
version: "3"
services:
  easyconnect:
    image: hagb/docker-easyconnect:cli
    container_name: docker-easyconnect
    environment:
      - EC_VER=7.6.7
      - IPTABLES_LEGACY=1
      - CLI_OPTS="-d https://vpn.hitsz.edu.cn -u username -p password"
    ports:
      - "1080:1080"
    volumes:
      - ./.easyconn:/root/.easyconn
    cap_add:
      - NET_ADMIN
    devices:
      - "/dev/net/tun:/dev/net/tun"
```
记得替换CLI_OPTS环境变量中的username和password。别的学校可替换`https://vpn.hitsz.edu.cn`后使用。

在easyconnect目录下执行：`docker-compose up -d`部署easyconnect镜像。停止容器`docker-compose stop`，删除容器并且删除网络`docker-compose down`。

于是就有一个在wsl中的代理`0.0.0.0:1080`。
```bash
ip addr show eth0 | grep 'inet ' | cut -f 6 -d ' ' | cut -f 1 -d '/'
```
执行上述命令获取wsl的ip。于是可在win下使用`wslip:1080`作为访问内网的代理。

使用代理的方法同[README](./README.md)。

> 仓促成文，若有疑问请提[issue](https://github.com/SoraShu/easyconn-socks5-for-HITsz/issues/new)。