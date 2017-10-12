---
layout: post
title:  "搭建个人ss服务"
date:   2017-10-12 10:35:21 +0800
categories: jekyll update
---

最近lantern不能上,shadowsock也不能注册了,于是只能自己想办法了......

## 第一步:购买VPS

我用的是Vultr,它是按小时收费的,下面是它的资费标准:

|费用/月|CPU| 内存|SSD|流量/月|
|:-------:|:-------:|:------:|:------:|:-------:|

|$2.5|1|512M|20G|500G|

|$5|1|1G|25G|1000G|

|$10|1|2G|40G|2000G|

|$20|2|4G|60G|3000G|

|$40|4|8G|100G|4000G|

|$80|6|16G|200G|5000G|

|$160|8|32G|300G|6000G|

|$320|16|64G|400G|10000G|

|$640|24|96G|800G|150000G|

目前那个$2.5售空了,我买的是$5/月的

下面先要充值,使用paypat绑定信用卡或是储蓄卡,支付宝充值时被退回来了,查了支付宝不再和他们合作了,首冲好像得要$10(大概66RMB).
![屏幕快照 2017-10-12 上午8.50.32.png](http://upload-images.jianshu.io/upload_images/7075902-21b2b684686ab940.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

充值之后

![屏幕快照 2017-10-12 上午9.11.58.png](http://upload-images.jianshu.io/upload_images/7075902-a283866065e45df8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![屏幕快照 2017-10-12 上午9.14.55.png](http://upload-images.jianshu.io/upload_images/7075902-d60aec0627e6bde1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![屏幕快照 2017-10-12 上午9.15.38.png](http://upload-images.jianshu.io/upload_images/7075902-c51d63dbcecc8d86.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![屏幕快照 2017-10-12 上午9.24.14.png](http://upload-images.jianshu.io/upload_images/7075902-6595fc506d7f3d5a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
等服务器显示running状态就差不多了
然后点击服务器,会出现下面信息

![屏幕快照 2017-10-12 上午9.26.14.png](http://upload-images.jianshu.io/upload_images/7075902-1e1cfe582ce52a7a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在终端ping 你的服务器地址看一下连接状态,连接不好的话,销毁再重新选一个服务器地址

![屏幕快照 2017-10-12 上午9.36.14.png](http://upload-images.jianshu.io/upload_images/7075902-1fe37ec692e6f86b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


每个VPS账号可以购买两个额外的ip,每月$2
![屏幕快照 2017-10-12 上午9.29.59.png](http://upload-images.jianshu.io/upload_images/7075902-a27c64e9e6fa500d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 第二步:部署服务器

### Mac版:
Terminal命令终端输入 ssh root@vultr服务器ip
然后会提示你输入密码,让第一步的密码复制粘贴上

![屏幕快照 2017-10-12 上午11.28.18.png](http://upload-images.jianshu.io/upload_images/7075902-baf520db4d8e95d7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
进入`root@vultr: ~#`成功

结束后依次输入下面三行
-    wget --no-check-certificate https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks-libev-debian.sh

![屏幕快照 2017-10-12 上午11.30.50.png](http://upload-images.jianshu.io/upload_images/7075902-a27c03df9d011e7d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- chmod +x shadowsocks-libev-debian.sh
-  ./shadowsocks-libev-debian.sh 2>&1 | tee shadowsocks-libev-debian.log


![屏幕快照 2017-10-12 上午11.33.52.png](http://upload-images.jianshu.io/upload_images/7075902-0b4b6eeaa814787b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![屏幕快照 2017-10-12 上午11.36.37.png](http://upload-images.jianshu.io/upload_images/7075902-ae55bc6babad032e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![屏幕快照 2017-10-12 上午11.42.21.png](http://upload-images.jianshu.io/upload_images/7075902-6656c06873404b6a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![屏幕快照 2017-10-12 上午11.48.39.png](http://upload-images.jianshu.io/upload_images/7075902-2841dde4189dfafd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
记住这个第四步需要

### window版看[自建ss服务器教程](https://github.com/Alvin9999/new-pac/wiki/自建ss服务器教程)

## 第三步: 使用Google BBR加速(可选)
terminal依次输入
- wget --no-check-certificate [https://github.com/teddysun/across/raw/master/bbr.sh](https://github.com/teddysun/across/raw/master/bbr.sh)

![屏幕快照 2017-10-12 上午11.51.02.png](http://upload-images.jianshu.io/upload_images/7075902-b440f019a07c3199.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- chmod +x bbr.sh
- ./bbr.sh

![屏幕快照 2017-10-12 上午11.52.05.png](http://upload-images.jianshu.io/upload_images/7075902-58904a464873dde4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![屏幕快照 2017-10-12 上午11.54.55.png](http://upload-images.jianshu.io/upload_images/7075902-e8460e45bf1402f6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
等个3~5分钟,选择重启就好了

## 第四部: Shadowsocks客户端下载

Mac: ShadowsocksX
window:好像是shadowsocksR

依次填入第二部设置的密码,端口(填:后面)
![屏幕快照 2017-10-12 上午10.01.30.png](http://upload-images.jianshu.io/upload_images/7075902-b673f047442177fa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
打开Google试一下就OK了.
