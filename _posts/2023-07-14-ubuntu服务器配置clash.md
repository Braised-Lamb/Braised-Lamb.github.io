---
layout: post
title:  "ubuntu服务器配置clash"
date:   2023-07-14 19:30:45 +0800
categories: ubuntu配置
tags: [ubuntu, clash]
toc: true
---

最近把实验室的服务器弄了一下，装了个clash，但是远程图形化桌面老是搞不定，查了一些资料实现了在web面板里调整设置

## 安装clash
- github仓库地址：`https://github.com/Dreamacro/clash/releases `

在合适位置新建文件夹`Clash`，使用命令行`mkdir`新建文件夹，本文使用目录为 `~/Clash`，在Clash文件夹里下载clash文件 
```bash
mkdir ~/Clash 
cd ~/Clash
wget -O https://github.com/Dreamacro/clash/releases/download/v1.17.0/clash-linux-386-v1.17.0.gz
```
解压文件，设置读写权限，重命名
```bash
gzip -d clash-linux-386-v1.17.0.gz #解压 
mv clash-linux-386-v1.17.0.gz clash #修改为clash
chmod +x Clash/clash #添加执行权限
```

## 配置

### clash配置

执行命令运行clash
```bash
./clash
```
初次运行会有提示下载初始配置文件在`~/.config/clash/`，包含`config.yaml`和`Country.mmdb`
提示如下：
```terminal
INFO[0000] Can't find config, create a initial config file 
INFO[0000] Can't find MMDB, start download              
INFO[0002] Mixed(http+socks) proxy listening at: 127.0.0.1:7890 
```
要配置自己的配置文件，在`clash`文件夹里下载订阅的`config.yaml`
```bash
wget -O <clash订阅地址>
```

### 代理配置

在使用clash的时候，考虑到诸如git、conda、python等环境都有可能使用代理，直接进行全局代理的设置，由clash来决定代理规则
在`~/.bashrc`末尾添加语句
```bash
# clash proxy
alias setProxy='export http_proxy=http://127.0.0.1:7890;export https_proxy=http://127.0.0.1:7890;export HTTP_PROXY=http://127.0.0.1:7890;export HTTPS_PROXY=http://127.0.0.1:7890'
alias unsetProxy='unset http_proxy;unset https_proxy;unset HTTP_PROXY;unset HTTPS_PROXY'
$(setProxy)
```
将`setProxy `注册为设置代理，`unsetProxy`为取消设置，可以直接在shell中运行
`$(setProxy)`表示在`.bashrc`文件里执行设置代理

使用`env | grep proxy`和`env | grep PROXY`验证设置情况

## 可视化面板
在`~/.config/clash/`文件夹里下载`gh-pages`仓库编译好的文件
```bash
git clone -b gh-pages https://github.com/Dreamacro/clash-dashboard ui
```
同时在`config.yaml`里设置端口和密钥
```yaml
# A RESTful API for clash
external-controller: <host_ip>:<port>
external-ui: ui

# Secret for RESTful API (Optional)
secret: "<secret>"
```
其中，`<host_ip>`修改为合适的ip，`<port>`设置为合适的端口，如果本地使用，用`127.0.0.1`，如果希望在局域网访问，就用服务器在局域网内的ip
在`http://yacd.haishan.me/`填写配置访问面板，URL使用`http://<host_ip>:<port>`
![image.png](https://cdn.jsdelivr.net/gh/Braised-Lamb/picbed/202307152011152.png)

连接成功后可以访问面板：
![image.png](https://cdn.jsdelivr.net/gh/Braised-Lamb/picbed/202307152011163.png)

## 启动方式一：注册服务

完成以上工作后，已经实现了clash的安装和配置，美中不足的地方在于，这样在终端启动的方法，需要保持前台活跃，也就是要一直留着一个窗口给clash
可以将clash注册为服务，实现开机启动和后台运行，新建文件
```bash
vim /etc/systemd/system/clash.service
```
内容填充为：
```
[Unit] 
Description=clash service 
After=network.target 

[Service] 
Type=simple 
User=root # 用户名
ExecStart=~/Clash/clash # clash文件路径 
Restart=on-failure # or always, on-abort, etc 

[Install] 
WantedBy=multi-user.target
```
设置开机自启
```bash
systemctl daemon-reload 
systemctl enable clash
```

之后也可以用以下语句启动clash
```bash
service clash start
```

## 启动方式二：.bashrc文件配置

在`.bashrc`文件里添加以下语句
```bash
alias clashOn='nohup ~/Clash/clash > /dev/null 2>&1 &'
alias clashOff='kill $(pgrep -f clash)'
$(clashOn)
```
`clashOn`实现后台启动clash
`clashOff`实现停止clash进程
可以通过`ps -ef | grep clash`来验证clash服务运行情况

## 参考
[在 Linux 服务器上安装 Clash，以及开机自动启动](https://www.xxpyy.top/detailed?id=14)

[服务器上配置clash - 真是古得](https://www.duckflew.cn/archives/fu-wu-qi-shang-pei-zhi-c-l-a-s-h)