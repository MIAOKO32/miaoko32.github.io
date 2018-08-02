---
layout:     post
title:      "自购VPS部署Shadowsocks简记"
subtitle:   " 自己动手简易够用梯子梯子 "
date:       2018-08-02
author:     "MIAOKO"
header-img: "img/google-miaoko.png"
catalog: true
tags:
    - 简记
    - 备忘
---

> 到了青岛，为了log in StackOverflow ，看到教程这么复杂，决定还是造个梯子吧

## 题外话
就在写文章的时候，Google一下markdown语法，又打不开了，以为是服务器又“宕机”了还是怎样，网上一搜，好多 **Vultr** 的服务器好像都有这样的问题，也有说跟运营商有关的，我滴个乖乖，以为自己10刀打水漂了，自己退了移动的校园网，连上实验室的Wi-Fi，ipv6地址ping的通了，Google能用了，但愿能就这样用着吧。



---

## 正文

### Vultr

最终选的[Vultr][www.vultr.com]，看了很多推荐，还有搬瓦工之类，To be honest，我是冲着¥2.5选的 **Vultr** ，再贵有点超出我的需求价值，或者就可以自己去买服务了，还能少花些经历。
<br>
<br>注册流程没什么说的
br1.create account
2.Billing：Paypal之类的可以灵活充值，支付宝需要最低支付$10,我估计是因为马云爸爸要的手续费挺多
3.deploy：选Vultr Cloud Computer ->
<br>以下是我的选择
|:--------:|:---------:|
|Location|Tokyo|
|Server Type|Debian 9|
|Server Size |$2.5|
|Others|\|

<br>$2.5的套餐，只有一个ipv6，绑定一个ipv4要$3,坑的不谈。
<br>选完之后会有一个 **installing** 的过程，完成后服务器就开始运行了。
`ping6 your server's IP address`
<br>接下来就是服务器端的部署了。

### SSH连接服务器
Mac 的 **Terminal** 支持直接建立远程连接，Shell -> New Remote Connection(shift+cmd+k).
<br>当然也可以在终端里直接
`ssh -p22 root@6217:5846:1234:678:5200:0000:8845:7777`
<br>选YES，把提供的密码粘贴进去。就可以进行操作了：
`root@vultr:~#`

### Shadowsocks-libev
之前参考的文章用的多是Shadowsocks的原版，因为 **某些原因** 仓库已经清空了，发现libev这个版本，看介绍好像是作者用C++写的，更轻量。
<br>首先安装 **Shadowsocks-lib** （Debian 9）其他系统可以去[项目仓库][https://github.com/shadowsocks/shadowsocks-libev].
<br>执行以下命令。
````
sh -c 'printf "deb http://deb.debian.org/debian stretch-backports main" > /etc/apt/sources.list.d/stretch-backports.list'
apt update
apt -t stretch-backports install shadowsocks-libev
````
<br>一切顺利的话，Shadowsocks就装好了。接下来就是SS-l的配置。
1.修改自己的配置文件。
2.将SS-l加入服务。
<br>
<br>SS-l的配置文件在`/etc/shadowsocks-libev/config.json`,因为这个目录下只有这一个文件，嫌麻烦直接`*.json`也ok。
<br>用 **VIM** 打开配置文件，`vi /etc/shadowsocks-libev/config.json`。
```
{
    "server":"::",
    "server_port":8388,
    "local_address": "127.0.0.1",
    "local_port":1080,
    "password":"mypassword",
    "timeout":300,
    "method":"aes-256-cfb",
    "fast_open": false
}
```
<br> `i`是进入编辑，`ESC`退出编辑接受指令，`:wq`保存并退出 **VIM**；
<br>
<br>此时运行`ss-server -6 -c /etc/shadowsocks-libev/config.json`应该已经可以了。
<br>
<br>为了在后台运行，直接运行默认的服务报错，不知道是不是Debian的问题，没仔细研究
<br>参照其他人方法，自己写了脚本，直接用脚本执行命令。步骤如下：
<br>新建用于启动服务的脚本。`vim /root/autoss.sh`,内容如下：
```
    #！/bin/bash
    #shadowsocks.sh
    ss-server -6 -c /etc/shadowsocks-libev/config.json
```
<br>`:wq`之后修改一下权限，`chmod 777 /root/autoss.sh`
<br>此时`./autoss.sh`,应该能启动服务，此时`opt+d`断开服务，SS-L服务应该还在运行。
<br>最后，想加入自启动中可以自己加进去，我没加因为之后可能还有改动。


###参考
[SS-l的仓库][https://github.com/shadowsocks/shadowsocks-libev]
[burning-yang][https://blog.csdn.net/ynb19930428/article/details/79078362]
[polarxiong][https://www.polarxiong.com/archives/Ubuntu-16-04下Shadowsocks服务器端安装及优化.html]
[过了即是客][https://blog.csdn.net/u011054333/article/details/52496303]
