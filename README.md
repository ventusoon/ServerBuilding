# LuvSia 
# 从零开始的服务器搭建纪实

准备工作：
VPS、解析到cloudflare的域名

```bash
#更新软件源
apt update
```
## 工具篇
### 宝塔面板 
1.使用一键配置工具，选择29→62
```bash
wget --no-check-certificate https://raw.githubusercontent.com/jinwyp/one_click_script/master/trojan_v2ray_install.sh && chmod +x ./trojan_v2ray_install.sh && ./trojan_v2ray_install.sh
```
2.安装nginx-1.21、mysql-5.5、php-7.4、phpmyadmin-5.0四件套

### X-UI
1.继续使用一键配置工具，安装X-UI，选择29→9
```bash
./trojan_v2ray_install.sh
```


