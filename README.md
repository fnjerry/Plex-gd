# Plex-gd
自动化：快速搭建基于GD的Plex私人媒体库  
自动化：快速搭建基于GD的Plex私人媒体库 - 基于Docker的Plex+Nginx+SSL自动安装配置脚本



介绍
自动安装Rclone、Docker，自动部署Plex服务端、Nginx服务端并自动签发SSL证书。

Nginx服务端自动对Plex服务端进行反带，使用自定义域名即可通过HTTPS对Plex进行访问，还可以将此域名直接接入CF进行加速。

系统要求
本文以Debian/Ubuntu系统为例

VPS：带宽最好大于100MB，性能不做要求

域名：用于CDN加速，可先对vps IP地址进行绑定

CF账号：选配项，用于CDN加速

使用方法
 1   sudo curl https://raw.githubusercontent.com/fnjerry/Plex-gd/main/plex.sh | sudo bash
 运行以上脚本，进行自动化安装。

配置Docker Compose文件
运行脚本之后，您还需要完成以下配置：

﻿1、设置域名DNS解析到本机IP

2、修改 docker-compose.yml 文件，确保正确设置FQDN-域名及邮箱地址 CERTBOT_EMAIL

3、将Rclone磁盘映射至/plex/disk的子目录下，例如：/plex/disk/gd

启动Plex容器
以上准备工作完成后，输入 docker-compose up -d 启动 plex docker 容器。

配置Plex
Plex启动之后，需要通过ssh端口映射到本机才能进入管理员配置界面，将vps的32400端口映射至127.0.0.1:32400端口，用浏览器打开127.0.0.1:32400：

 

设置Plex的别名、媒体库设置可以先跳过。



设置完成后，点击进入到Plex服务器主界面，记得登录自己的Plex账号，并声明服务器。

配置Plex外部访问及关闭官方中继加速（减速）
进入Plex服务器设置，确保开启“显示高级设置”。

进入“网络”选项卡，滚动到页尾：

找到“Custom server access URLs”，添加自定义访问URL，例：http://<vps ip>:80 ，如果已绑定cf域名还可输入域名地址，多个网址使用英文,进行分隔（端口记得于docker设置中保持一致）

取消勾选"Enable Relay"，将停用Plex官方的中继加速服务，因为我们有独立IP且中继服务实际有带宽限制，实际对我们来说是减速行为，这里要关掉。

 

进入“远程访问选项”卡

点击“禁用远程访问”，由于已经设置了外部访问的IP和域名，这里我们将阻止所有其他的访问，避免类似中继对访问Plex服务照成负面影响

配置Plex转码加速
进入“转码器”选项卡

性能一般：勾选“Disable video stream transcoding”。例如只有1～2核的情况，建议关闭视频转码，使用原码率观看影片，只要带宽足够，实测在播放20G的文件时也很流畅。
性能不错：勾选“Use hardware acceleration when available”及“Use hardware-accelerated video encoding”，开启硬件加速。
添加媒体库
（略）

开启Plex新世界的大门
现在，使用http://<ip或域名>直接访问Plex的服务，使用Web端或App就可以看到自己的Plex服务器了，播放视频速度也会快上许多。

Plex手机端APP设置
推荐使用APP进行观看，因为关闭转码之后，很多视频编码格式浏览器无法识别，使用网页端的Plex播放会提示没有足够的资源进行转码。

 

如果服务端关闭了视频转码，Plex APP也需要关闭转码功能，设置方法：

设置>质量，关闭“自动调整质量”，将“Remote streaming quality”设置为“最高的”，将“家庭串流”设置为“最高的”，取消“在移动网络使用低质量”选项。

进阶篇：基于Google Drive的Plex私人媒体库使用Nginx进行中继的方法
此教程小白劝退，废话少说，开搞！

 

系统要求
本文以Debian/Ubuntu系统为例

VPS：带宽最好大于100MB，性能不做要求

CF账号：选配项，用于CDN加速

域名：选配项，用于CDN加速，可先对vps地址进行绑定并开启CF加速

 

 配合上一篇教程，在docker命令需要调整端口。

Plex安装配置
 安装
输入以下命令来建立一个plex的docker容器：

docker run -d \
  --name=plex \
  -e PUID=0 \
  -e PGID=0 \
  -e VERSION=docker \
  -e UMASK_SET=022 \
  -e TZ="Asia/Shanghai" \
  -e PLEX_CLAIM=这里输入自己的CLAIM来绑定 \
  -v /root/plex/config:/config \
  -v /root/plex/transcode:/transcode \
  -v /drive:/drive \
  -p 127.0.0.1:32400:32400 \
  --restart unless-stopped \
  --device=/dev/dri:/dev/dri \
  ghcr.io/linuxserver/plex
-p 127.0.0.1:32400:32400  映射plex服务32400端口到主机的相同端口，只允许内部访问

-e PLEX_CLAIM  这是Plex服务端自动绑定账号的密钥，可以在官方地址获取

-v /root/plex/config:/config  映射plex数据库到本地（左侧为本机路径可自主修改）

-v /root/plex/transcode:/transcode  映射plex转码目录（左侧为本机路径可自主修改）

-v /drive:/drive  映射本地rclone挂载目录到本地（建议将所有gd挂载盘作为driver的子目录，这样增删云盘的挂载就不需要重启docker容器啦）

Nginx安装配置
前期工作
安装Nginx并设置好Host主机及SSl证书
添加反向Dai理，映射到http://127.0.0.1:32400
重点
反Die需要单独指定以下配置项，否则会产生诸如：WebSocket失联导致后台无法正常显示服务器网速及负载等信息；部分Header标头缺失导致部分H264编码mp4文件长时间缓冲且无法播放的问题。

下面给出解决方案的配置项：

    # Plex start
    # 解决视频预览进度条无法拖动的问题
    proxy_set_header Range $http_range;
    proxy_set_header If-Range $http_if_range;
    proxy_no_cache $http_range $http_if_range;
    
    # 反带流式，不进行缓冲
    client_max_body_size 0;
    proxy_http_version 1.1;
    proxy_request_buffering off;
    #proxy_ignore_client_abort on;
    
    # 同时反带WebSocket协议
    proxy_set_header X-Forwarded-For $remote_addr:$remote_port;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection upgrade;
    # Plex end
如果你要使用更快的前端来反Die建议中间套上CF，结构为Plex VPS > CF > Front Server，如果前端服务器的位置不错，会有很可观的提速功能，既节省了前端的性能，将负载较重的任务留给后端服务器来处理，性能与速度兼备。
快速搭建基于Google Drive的Plex私人媒体库（附加CDN提速方法）
准备工作
系统要求
本文以Debian/Ubuntu系统为例

VPS：带宽最好大于100MB，性能不做要求

CF账号：选配项，用于CDN加速

域名：选配项，用于CDN加速，可先对vps地址进行绑定并开启CF加速

安装Docker
运行以下代码进行安装：

# 安装docker
sudo apt-get update -y
curl -fsSL get.docker.com -o get-docker.sh
sudo sh get-docker.sh
安装Rclone
curl https://rclone.org/install.sh | sudo bash
# 安装fuse 支持
sudo apt-get install fuse -y
sudo apt-get install nohup -y
rclone安装后需要运行rclone config及添加google drive云盘，也可以直接复制已有的配置项到/<用户目录>/.config/rclone/rclone.conf文件中。

使用Rclone映射Google Drive
建议使用以下命令：

sudo nohup rclone mount gd: /drive/gd --cache-dir /tmp --vfs-cache-mode writes --no-gzip-encoding --no-check-certificate --allow-non-empty --allow-other --umask 0000 &
gd:  替换为rclone中配置的名称

/drive/gd  替换为需要映射GD到本地的目录位置（记得先创建好这个目录）

Plex安装配置﻿
准备工作做好之后，我们这里选用了Docker版的plex进行安装，优点是简单、快速、方便调试。

安装
输入以下命令来建立一个plex的docker容器：

docker run -d \
  --name=plex \
  -e PUID=0 \
  -e PGID=0 \
  -e VERSION=docker \
  -e UMASK_SET=022 \
  -e TZ="Asia/Shanghai" \
  -e PLEX_CLAIM=这里输入自己的CLAIM来绑定 \
  -v /root/plex/config:/config \
  -v /root/plex/transcode:/transcode \
  -v /drive:/drive \
  -p 80:32400 \
  --restart unless-stopped \
  --device=/dev/dri:/dev/dri \
  ghcr.io/linuxserver/plex
-p 80:32400  映射plex服务32400端口到主机的80端口，也可以根据需要自行指定

-e PLEX_CLAIM  这是Plex服务端自动绑定账号的密钥，可以在官方地址获取

-v /root/plex/config:/config  映射plex数据库到本地（左侧为本机路径可自主修改）

-v /root/plex/transcode:/transcode  映射plex转码目录（左侧为本机路径可自主修改）

-v /drive:/drive  映射本地rclone挂载目录到本地（建议将所有gd挂载盘作为driver的子目录，这样增删云盘的挂载就不需要重启docker容器啦）

配置Plex
安装好之后，需要通过ssh端口映射到本机才能进入管理员配置界面。

端口映射可使用putty (Windows)、Termius (Android) 。

以Putty为例的映射方法：
打开putty，在connection – S*S*H – Tunnels下设置source port 8888， destination 127.0.0.1:32400，然后点击Add。

Plex添加端口

以Termius为例的映射方法：
打开Termius，点击左上角按钮展开菜单，选择Port forwarding，点击右下角+号，Host from选择VPS主机、Port from输入32400、Host to输入localhost、Port to输入32400、Address输入127.0.0.1。

点击putty登录，打开浏览器访问http://127.0.0.1:32400/web，就可以看到Plex的Web界面了。

设置Plex的别名、媒体库设置可以先跳过。



设置完成后，点击进入到Plex服务器主界面，记得登录自己的Plex账号，并声明服务器。

 
配置Plex外部访问及关闭官方中继加速（减速）
进入Plex服务器设置，确保开启“显示高级设置”。

进入“网络”选项卡，滚动到页尾：

找到“Custom server access URLs”，添加自定义访问URL，例：http://<vps ip>:80 ，如果已绑定cf域名还可输入域名地址，多个网址使用英文,进行分隔（端口记得于docker设置中保持一致）

取消勾选"Enable Relay"，将停用Plex官方的中继加速服务，因为我们有独立IP且中继服务实际有带宽限制，实际对我们来说是减速行为，这里要关掉。

 

进入“远程访问选项”卡

点击“禁用远程访问”，由于已经设置了外部访问的IP和域名，这里我们将阻止所有其他的访问，避免类似中继对访问Plex服务照成负面影响

配置Plex转码加速
进入“转码器”选项卡

性能一般：勾选“Disable video stream transcoding”。例如只有1～2核的情况，建议关闭视频转码，使用原码率观看影片，只要带宽足够，实测在播放20G的文件时也很流畅。
性能不错：勾选“Use hardware acceleration when available”及“Use hardware-accelerated video encoding”，开启硬件加速。
添加媒体库
（略）

开启Plex新世界的大门
现在，使用http://<ip或域名>直接访问Plex的服务，使用Web端或App就可以看到自己的Plex服务器了，播放视频速度也会快上许多。

Plex手机端APP设置
推荐使用APP进行观看，因为关闭转码之后，很多视频编码格式浏览器无法识别，使用网页端的Plex播放会提示没有足够的资源进行转码。

 

如果服务端关闭了视频转码，Plex APP也需要关闭转码功能，设置方法：

设置>质量，关闭“自动调整质量”，将“Remote streaming quality”设置为“最高的”，将“家庭串流”设置为“最高的”，取消“在移动网络使用低质量”选项。
 
 
