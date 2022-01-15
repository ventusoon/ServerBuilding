# LuvSia 
# 从零开始的服务器搭建纪实

一、准备工作：

VPS、解析到cloudflare的域名

```bash
#更新软件源
apt update
```
二、搭建思路：

预留主站点域名地址给网站搭建，给各个应用添加前端路径。

#-若显示“不安全连接”，再次在宝塔的SSL中开启“开启强制使用https”。

### 一、宝塔面板 
1.使用一键配置工具，选择29→62
```bash
wget --no-check-certificate https://raw.githubusercontent.com/jinwyp/one_click_script/master/trojan_v2ray_install.sh && chmod +x ./trojan_v2ray_install.sh && ./trojan_v2ray_install.sh
```
2.安装nginx-1.21、mysql-5.5、php-7.4、phpmyadmin-5.0四件套

3.添加站点。

4.设置，添加SSL，开启强制使用https。

5.安装docker安装器。

### 二、X-UI on docker

1.添加镜像源
```bash
enwaiax/x-ui:latest
```
2.不配置容器，使用代码
```bash
mkdir x-ui && cd x-ui
docker run -itd --network=host \
    -v $PWD/db/:/etc/x-ui/ \
    -v $PWD/cert/:/root/cert/ \
    --name x-ui --restart=unless-stopped \
    enwaiax/x-ui:latest
```
3.进入x.x.x.x:54321，修改X-UI面板路径 /x

4.添加反向代理，开启高级功能
```bash
代理名称 x-ui
代理目录 /x
目标URL  http://127.0.0.1:54321  发送域名 $host
#其余留空
```

### 三、Alist on docker

1.添加镜像源
```bash
xhofe/alist:latest
```
2.不配置容器，使用代码
```bash
docker run -d --restart=always -v /etc/alist:/opt/alist/data -p 5244:5244 --name="alist" xhofe/alist:latest
```

3.进入x.x.x.x:5244，添加alist路径 /a

4.添加反向代理，开启高级功能
```bash
代理名称 alist
代理目录 /a
目标URL  http://127.0.0.1:5244  发送域名 $host
#其余留空
```



