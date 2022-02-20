# easyconn-socks5-for-HITsz

在服务器上运行easyconnect并建立socks5代理，最终实现win电脑上免安装easyconnect访问校园内网。

## easyconnect 从下载到放弃

不知道是软件太老还是其他什么原因（鬼知道），easyconnect这软件在桌面端屡屡显露出一些流氓特性。包括但不限于：

1. 开机自启动的服务。服务大致长`Sangfor***`的样子。关也关不掉，taskkill之后又会再生。当然它作为service的子进程自启也是合理的。但我相信大多数人都不是每次开机后都需要使用这个软件。
2. 卸载之后上述服务不会被删除，时不时在你玩游戏或者关机的时候给你惊喜。
3. 自动安装的CA证书。该CA证书应该是起到了让你通过它访问网站的时候浏览器不会“跳不安全的站点”，但安装CA证书却不告知用户。
4. 对深信服这个公司并不信任。关于深信服的黑料可自行搜索。

受 [HITsz-daily-report][1],以及项目中使用的 [docker-easyconnect][2] 的启发，并且在某乎上找到了相关专栏，我决定将 easyconnect 放在服务器里跑，docker 干净又方便，然后配置socks5代理。

## 简单上手（云服务器版）

首先需要有一台云服务器。无服务器的win用户可以使用wsl起容器。见[本地docker方案](./本地docker方案.md)。

~~自己折腾是不可能的，只会用别人写好的，才能维持得了生活这样子~~

[docker-easyconnect][2] 项目不止是将 easyconnect 放进 docker 里，还提供了 socks5 服务。所有的网络通信都被隔离在容器了，并且只暴露了一个 socks5 端口出来当代理端口。

ssh 登录服务器后，在 bash 中执行：

```bash
touch ~/.easyconn
echo -e "https://vpn.hitsz.edu.cn\n%username\n%password" > setting.conf
docker run --device /dev/net/tun --name 'EasyConnect' --cap-add NET_ADMIN -i -v $HOME/.easyconn:/root/.easyconn -p 127.0.0.1:1080:1080 -e EC_VER=7.6.7 hagb/docker-easyconnect:cli < setting.conf &
```
> 第二行的`%username`与`%password`自行替换为统一认证用户名与密码，别校的vpn请自行替换前面的地址。  
> 直接copy了[HITsz-daily-report][1]的代码。  
> :warning:若vpn地址在后面多加了一条斜杠则会产生`svpn stop!`,`auto loggin success!`循环。

此时执行`lsof -i tcp:1080`就能看到已经有一个socks5代理跑在服务器上了。端口是1080。~~接下去就把这个端口暴露在公网下就可以了。~~

直接暴露端口肯定有安全问题，似乎可以弄鉴权，太麻烦就不弄。使用ssh端口转发。这一步请务必退出ssh或者新开一个终端操作。

```batch
ssh -L 1080:127.0.0.1:1080 root@yourIP
```
自行替换 `yourIP` 为服务器 IP，相当于在 ssh 登录的同时设置了一个端口转发，把服务器的 1080 端口转发到本地的 1080 端口。

关于如何使用这个socks5代理，个人一直是用 clash，用 clash for windows 或者其他 GUI 都可。我个人是用不到校园内网的远程桌面和内网ssh，有需求的自行研究 clash 的 tun 模式。就上个教务网啥的普通配置就行。

clash的配置文件示例：

```yaml
# 代理名为vpn
proxies:
 - {"name": "vpn", "type": "socks5", "server": "127.0.0.1", "port": "1080"}

# 设置以hitsz.edu.cn为域名后缀的连接走代理
rules:
 - DOMAIN-SUFFIX,hitsz.edu.cn,vpn
```
将此yaml文件拖入clash中，切换到此配置文件，开启规则模式和系统代理，就可以访问以hitsz.edu.cn为域名后缀的网站了，同时其他网站不受影响。

关于规则的配置还会有其他需求，比如上知网，上图书馆官网，以后碰到了再现写。

具体规则写法：[clash规则编写](https://docs.cfw.lbyczf.com/contents/ui/profiles/rules.html)

## 体验

不知道是不是错觉，感觉绕了一大弯子访问内网反而变快了，真是人间迷惑。

## 卸载 easyconnect

见[uninstall the fucking easyconnect](uninstall-the-fucking-easyconnect.md)

> 仓促成文，若有疑问请提[issue](https://github.com/SoraShu/easyconn-socks5-for-HITsz/issues/new)。

[1]:https://github.com/JalinWang/HITsz-daily-report
[2]:https://github.com/Hagb/docker-easyconnect