LuvSia 
从零开始的服务器搭建纪实
===========================
# 目录
* [摩拳擦掌](#准备工作)
* [开阔思路](#搭建思路)
* [废寝忘食](#开始搭建)
    * [宝塔面板](#宝塔面板)
    * [X-UI on docker](#x-ui-on-docker)
    * [Alist on docker](#alist-on-docker)
    * [Transmission on docker](#Transmission-on-docker)
    * [Rclone mount GoogleDrive](#Rclone-mount-GoogleDrive)
    * [Cloudflare WARP](#Cloudflare-WARP)
* [朝花夕拾](#后记)
* [巨人之肩](#感谢)

# 准备工作
|VPS|Domain||||||
|---|:---:|:----:|:---:|:----:|:---:|:----:|
|[<img src="https://github.com/ventusoon/LuvSia/raw/main/logo/dmit.png" width="200px">](https://www.dmit.io/)|example.com|[<img src="https://github.com/ventusoon/LuvSia/raw/main/logo/nginx.svg" width="100px">](https://www.nginx.com)|[<img src="https://github.com/ventusoon/LuvSia/raw/main/logo/mysql.png" width="80px">](https://www.mysql.com)|[<img src="https://github.com/ventusoon/LuvSia/raw/main/logo/php.svg" width="70px">](https://www.php.net)|[<img src="https://github.com/ventusoon/LuvSia/raw/main/logo/phpmyadmin.png" width="100px">](https://www.phpmyadmin.net)|[<img src="https://github.com/ventusoon/LuvSia/raw/main/logo/cloudflare.svg" width="150px">](https://www.cloudflare.com)|

*更新软件源*
```
apt update
```
# 搭建思路

**预留一级域名搭建网站**，**~~给各个应用添加前端网页根路径。~~** 

**后因部分应用无法添加`二级目录`（网页根路径），所以这里只能使用添加`二级域名`的思路，区别各个应用。** 

**再通过添加`反向代理`，实现分域名访问不同前端应用。**

# 开始搭建

## `宝塔面板`
|Work|Web|
|:----:|:---:|
|宝塔面板|[<img src="https://github.com/ventusoon/LuvSia/raw/main/logo/bt.png" width="65px">](https://www.hostcli.com)|


**1.使用一键配置工具。**
一键脚本集成工具
```bash
wget --no-check-certificate https://raw.githubusercontent.com/jinwyp/one_click_script/master/trojan_v2ray_install.sh && chmod +x ./trojan_v2ray_install.sh && ./trojan_v2ray_install.sh
```
或
```
wget -O install.sh http://download.yu.al/install/install-ubuntu_6.0.sh && bash install.sh
```

**2.安装`nginx-1.21`、`mysql-5.5`、`php-7.4`、`phpmyadmin-5.0`四件套**

**3.添加站点。**
|Work|Domain||
|---|---|:---:|
|alist|a.example.com|**正常**|
|宝塔面板|b.example.com|**正常**|
|Transmission|t.example.com|**正常**|
|X-UI|x.example.com|**正常**|

**4.设置，添加SSL，`强制开启https`。**

**记录`证书路径`，证书可以直接在`宝塔面板`进行更新，或是设置定时任务自动更新;**

**`宝塔面板`申请的证书在如下目录：`/www/server/panel/vhost/cert/你的域名/` 目录之下。**  

***`强制开启https`***。

**5.关闭面板`安全入户`，即删除`二级目录`。**
```c
rm -f /www/server/panel/data/admin_path.pl
```

**6.在`Cloudflare`中解析二级域名`b.example.com`**

**7.添加反向代理到`b.example.com`**
```
代理名称 宝塔面板
目标URL  http://127.0.0.1:8888  发送域名 $host
```
**8.安装`docker`安装器。**

## `X-UI` on `docker`
|Work|Web|Tools|
|---|---|---
|x-ui|[<img src="https://github.com/ventusoon/LuvSia/raw/main/logo/xui.png" width="65px">](https://github.com/vaxilu/x-ui)|[<img src="https://github.com/ventusoon/LuvSia/raw/main/logo/docker.png" width="65px">](https://hub.docker.com/r/enwaiax/x-ui)

**1.添加镜像源**
```
enwaiax/x-ui:latest
```
**2.不配置容器，使用代码**
```bash
mkdir x-ui && cd x-ui
docker run -itd --network=host \
    -v $PWD/db/:/etc/x-ui/ \
    -v $PWD/cert/:/root/cert/ \
    --name x-ui --restart=unless-stopped \
    enwaiax/x-ui:latest
```

**3.在`Cloudflare`中解析二级域名`x.example.com`**

**4.添加反向代理到`x.example.com`**
```
代理名称 x-ui
目标URL  http://127.0.0.1:54321  发送域名 $host
```

**5.配置`VMess`协议，开启`ws`，路径`/xiya`。**

**6.开启CDN加速，在`nginx`配置文件中添加如下。**
```javascript
location /xiya {
        proxy_redirect off;
        proxy_pass http://127.0.0.1:22513;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $http_host;
        proxy_read_timeout 300s;
        # Show realip in v2ray access.log
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  }
```

**7.开启`Cloudflare`小云朵，并优选IP。**

**8.[下载优选IP](https://github.com/XIU2/CloudflareSpeedTest/releases) ，选出优选IP后，客户端如下配置**
```diff
-填入优选IP
-更改端口为443；
-host设置x.example.com；
-开启tls。
```

**9.Surge配置**
```java
🇭🇰 VMess = vmess, 104.19.79.223, 443, username=2d285385-836d-4b30-f32c-11cdd637aeed, tls=true, tls13=false, ws=true, ws-path=/xiya, sni=example.com, ws-headers=Host:example.com, skip-cert-verify=0, tfo=false, udp-relay=true
```

## `Alist` on `docker`
|Work|Web|Tools|
|---|---|---
|Alist| [<img src="https://github.com/ventusoon/LuvSia/raw/main/logo/alist.png" width="65px">](https://github.com/Xhofe/alist)|[<img src="https://github.com/ventusoon/LuvSia/raw/main/logo/docker.png" width="65px">](https://hub.docker.com/r/xhofe/alist)|

**1.添加镜像源**
```
xhofe/alist:latest
```
**2.不配置容器，使用代码**
```javascript
docker run -d --restart=always -v /etc/alist:/opt/alist/data -p 5244:5244 --name="alist" xhofe/alist:latest
```

**3.在`Cloudflare`中解析二级域名`a.example.com`**

**4.添加反向代理到a.example.com**
```
代理名称 alist
目标URL  http://127.0.0.1:5244  发送域名 $host
```

## `Transmission` on `docker`
|Work|Web|Tools|
|---|---|---
|Transmission| [<img src="https://github.com/ventusoon/LuvSia/raw/main/logo/transmission.png" width="150px">](https://github.com/helloxz/docker-transmission)|[<img src="https://github.com/ventusoon/LuvSia/raw/main/logo/docker.png" width="65px">](https://hub.docker.com/r/helloz/transmission)|

### 1.Docker安装

**构建镜像**
```
# 克隆仓库
git clone https://github.com/helloxz/docker-transmission.git
# 进入仓库目录
cd docker-transmission
# 构建Docker镜像
docker build -t luvsia:transmission
```
**运行镜像**
```javascript
docker run -d --name="transmission" \
  -e USERNAME=ventus \
  -e PASSWORD=ysw554247430 \
  -p 9091:9091 \
  -p 51413:51413 \
  -p 51413:51413/udp \
  -v /GoogleDrive:/Downloads \
  --restart=always \
  luvsia:transmission
```

**/GoogleDrive:本地下载目录为[`Rclone` mount `GoogleDrive`](#Rclone-mount-GoogleDrive)中的谷歌云盘目录，自动下载到谷歌云盘，实现无线网盘**

### 2.1普通安装

**安装transmission-daemon**
```c
apt-get install transmission-daemon
```
**首先执行一次启动和停止命令，防止配置文件被覆盖**

启动
```
service transmission-daemon start
```
停止
```
service transmission-daemon stop
```
重启
```
service transmission-daemon restart
```

**修改transmission-daemon配置文件**
```c
nano /var/lib/transmission-daemon/info/settings.json
```
```
"rpc-host-whitelist": "*",  //域名白名单，*为允许所有
"rpc-host-whitelist-enabled": false, //是否开启白名单，false为否
"rpc-password": "远程登录密码",
"rpc-port": 9091, //远程登录端口
"rpc-username": "远程帐号",
"rpc-whitelist": "*", //ip白名单
"rpc-whitelist-enabled": false,  //是否开启ip白名单，false为否
```
**执行启动命令**
```
service transmission-daemon start
```

### 2.2[transmission-web-control](https://github.com/ronggang/transmission-web-control/wiki/Linux-Installation-CN)美化

**获取安装脚本**
```
wget https://github.com/ronggang/transmission-web-control/raw/master/release/install-tr-control-cn.sh
```

**执行安装脚本**
```
bash install-tr-control-cn.sh
```

**3.在`Cloudflare`中解析二级域名`t.example.com`**

**4.添加反向代理到t.example.com**
```
代理名称 transmission
目标URL  http://127.0.0.1:9091  发送域名 $host
```

## `Rclone` mount `GoogleDrive`
|Work|Web|Tools|
|---|---|---
|Rclone| [<img src="https://github.com/ventusoon/LuvSia/raw/main/logo/rclone.svg" width="150px">](https://github.com/rclone/rclone)|[<img src="https://github.com/ventusoon/LuvSia/raw/main/logo/googledrive.png" width="65px">](https://drive.google.com)|

**1.安装`Rclone`**
```bash
wget https://www.moerats.com/usr/shell/rclone_debian.sh && bash rclone_debian.sh
```

**2.初始化配置**
```
rclone config
```

**3.选择`GoogleDrive`，剩余操作参考[这里](https://www.moerats.com/archives/491/)**

**4.挂载磁盘**
```
mkdir -p /GoogleDrive
```
```
mkdir -p /Downloads
```
```javascript
/usr/bin/rclone mount luvsia:ventus /GoogleDrive \
 --umask 0000 \
 --default-permissions \
 --allow-non-empty \
 --allow-other \
 --transfers 4 \
 --buffer-size 32M \
 --low-level-retries 200
 --dir-cache-time 12h
 --vfs-read-chunk-size 32M
 --vfs-read-chunk-size-limit 1G
 ```
**5.配置开机自动挂载**
```javascript
cat > /etc/systemd/system/rclone.service <<EOF
[Unit]
Description=Rclone
AssertPathIsDirectory=LocalFolder
After=network-online.target

[Service]
Type=simple
ExecStart=/usr/bin/rclone mount luvsia:ventus /GoogleDrive \
 --umask 0000 \
 --default-permissions \
 --allow-non-empty \
 --allow-other \
 --buffer-size 32M \
 --dir-cache-time 12h \
 --vfs-read-chunk-size 64M \
 --vfs-read-chunk-size-limit 1G
ExecStop=/bin/fusermount -u LocalFolder
Restart=on-abort
User=root

[Install]
WantedBy=default.target
EOF
```

**挂载成功后，输入`df -h`命令即可看到挂载的磁盘**

**6.常用命令**

```
systemctl start rclone
```
```
systemctl enable rclone
```
```
systemctl stop rclone
```
```
systemctl status rclone
```

**7.`Transmission`添加`rlone`挂载在`GoogleDrive`上的路径**
```
/GoogleDrive
```

## `Cloudflare WARP`
|Work|Web|Use|
|---|---|---
|Cloudflare WARP|[<img src="https://github.com/ventusoon/LuvSia/raw/main/logo/cloudflare.svg" width="150px">](https://github.com/P3TERX/warp.sh)|[使用教程](https://p3terx.com/archives/cloudflare-warp-configuration-script.html)|

**功能菜单**
```Bash
bash <(curl -fsSL git.io/warp.sh) menu
```
**`WARP WireGuard` 双栈全局网络**
```Bash
bash <(curl -fsSL git.io/warp.sh) d
```
**`IPv4` 网络**
```Bash
bash <(curl -fsSL git.io/warp.sh) 4
```
**`IPv6` 网络**
```Bash
bash <(curl -fsSL git.io/warp.sh) 6
```
**`WARP 官方客户端` `SOCKS5` 代理**
```Bash
bash <(curl -fsSL git.io/warp.sh) s5
```

# 后记
# 感谢
[@guodongxiaren](https://github.com/guodongxiaren/README)  
[@HostCLi](https://www.hostcli.com)  
[@jinwyp](https://github.com/jinwyp/one_click_script)  
[@Xhofe](https://github.com/Xhofe/alist)  
[@vaxilu](https://github.com/vaxilu/x-ui)  
[@enwaiax](https://hub.docker.com/r/enwaiax/x-ui)  
[@XIU2](https://github.com/XIU2/CloudflareSpeedTest)  
[@helloxz](https://github.com/helloxz/docker-transmission)  
[@moerats](https://www.moerats.com/archives/491)  
[@P3TERX](https://github.com/P3TERX/warp.sh)  


