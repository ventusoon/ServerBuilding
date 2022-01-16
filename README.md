LuvSia 
从零开始的服务器搭建纪实
===========================
# 目录
* [准备工作](#准备工作)
* [搭建思路](#搭建思路)
* [开始搭建](#开始搭建)
    * [宝塔面板](#宝塔面板)
    * [X-UI on docker](#x-ui-on-docker)
    * [Alist on docker](#alist-on-docker)
    * [Transmission on docker](#Transmission-on-docker)
    * [Rclone mount GoogleDrive](#Rclone-mount-GoogleDrive)
    * [Cloudflare WARP](#Cloudflare-WARP)


# 准备工作

VPS、解析到cloudflare的域名

*更新软件源*
```
apt update
```
# 搭建思路

**预留主站点域名地址给网站搭建**，**~~给各个应用添加前端路径~~**。

**部分应用无法使用二级目录，所以这里只能使用添加二级域名的思路，区别各个应用。** 

**记录证书路径，证书可以直接在宝塔面板进行更新，或是设置定时任务自动更新;**

**宝塔面板申请的证书在如下目录：/www/server/panel/vhost/cert/你的域名/ 目录之下。**  

***不要`强制开启https`***。

# 开始搭建

## 宝塔面板 
**1.使用一键配置工具，选择29→62**
```bash
wget --no-check-certificate https://raw.githubusercontent.com/jinwyp/one_click_script/master/trojan_v2ray_install.sh && chmod +x ./trojan_v2ray_install.sh && ./trojan_v2ray_install.sh
```
![](https://github.com/ventusoon/LuvSia/raw/main/img/sh.png)
**2.安装nginx-1.21、mysql-5.5、php-7.4、phpmyadmin-5.0四件套**
![](https://github.com/ventusoon/LuvSia/raw/main/img/bt.png)
**3.添加站点。**

**4.设置，添加SSL，开启强制使用https。**

**5.安装docker安装器。**

## X-UI on docker

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

**3.在Cloudflare中解析二级域名`x.example.com`**

**4.添加反向代理到`x.example.com`**
```
代理名称 x-ui
目标URL  http://127.0.0.1:54321  发送域名 $host
```

**5.配置`VMess`协议，开启`ws`，路径`/xiya`。**

**6.开启CDN加速，在nginx配置文件中添加如下。**
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

**7.开启Cloudflare小云朵，并优选IP。**

**8.[下载优选IP](https://github.com/ventusoon/LuvSia/raw/main/tools/%E4%BC%98%E9%80%89ip.zip) ，选出优选IP后，客户端如下配置**
```diff
-填入优选IP
-更改端口为443；
-host设置x.example.com；
-开启tls。
```

## Alist on docker
![Alist](https://store.heytapimage.com/cdo-portal/feedback/202112/05/1542f45f86b8609495b69c5380753135.png)
**1.添加镜像源**
```
xhofe/alist:latest
```
**2.不配置容器，使用代码**
```javascript
docker run -d --restart=always -v /etc/alist:/opt/alist/data -p 5244:5244 --name="alist" xhofe/alist:latest
```

**3.在Cloudflare中解析二级域名a.example.com**

**4.添加反向代理到a.example.com**
```
代理名称 alist
目标URL  http://127.0.0.1:5244  发送域名 $host
```

## Transmission on docker

**1.添加镜像源**
```
helloz/transmission
```
**2.不配置容器，使用代码**
```javascript
docker run -d --name="transmission" \
  -e USERNAME=ventus \
  -e PASSWORD=ysw554247430 \
  -p 9091:9091 \
  -p 51413:51413 \
  -p 51413:51413/udp \
  -v /data/downloads:/root/Downloads \
  -v /data/transmission:/root/.config/transmission-daemon \
  --restart=always \
  helloz/transmission
```

**3.在Cloudflare中解析二级域名t.example.com**

**4.添加反向代理到t.example.com**
```
代理名称 transmission
目标URL  http://127.0.0.1:9091  发送域名 $host
```

## Rclone mount GoogleDrive

**1.安装Rclone**
```bash
wget https://www.moerats.com/usr/shell/rclone_debian.sh && bash rclone_debian.sh
```

**2.初始化配置**
```
rclone config
```

**3.选择GoogleDrive，剩余操作参考[这里](https://www.jiyiblog.com/archives/031167.html)**

**4.挂载磁盘**
```
mkdir -p /GoogleDrive
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

**挂载成功后，输入df -h命令即可看到挂载的磁盘**

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

**7.Transmission添加rlone挂载在GoogleDrive上的路径**
```
/GoogleDrive
```

## Cloudflare WARP
[使用教程](https://p3terx.com/archives/cloudflare-warp-configuration-script.html)

**功能菜单**
```Bash
bash <(curl -fsSL git.io/warp.sh) menu
```
**WARP WireGuard 双栈全局网络**
```Bash
bash <(curl -fsSL git.io/warp.sh) d
```
**IPv4 网络**
```Bash
bash <(curl -fsSL git.io/warp.sh) 4
```
**IPv6 网络**
```Bash
bash <(curl -fsSL git.io/warp.sh) 6
```
**WARP 官方客户端 SOCKS5 代理**
```Bash
bash <(curl -fsSL git.io/warp.sh) s5
```
