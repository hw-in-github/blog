---
title: Nodejs项目线上发布之pm2部署hexo博客
date: 2019-04-08 20:03:04
tags: [devops, nodejs, pm2]
---

## 搭建Nodejs生产环境
首先安装一些必要的编译工具、开发工具
```shell
sudo apt-get install vim openssl build-essential libssl-dev wget curl git
```
## 安装nvm
有了nodejs版本管理，就可以轻易管理多个nodejs的版本，在github上找到`creationix/nvm`的安装版本
```shell
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.34.0/install.sh | bash
```

### 手动安装nvm
安装过程可能出现很多问题，可以选择手动安装
1. `cd ~/`切换到用户根目录，下载nvm, `git clone https://github.com/creationix/nvm.git .nvm`
2. 启动nvm，切换到nvm目录，source一下，`. nvm.sh`
### 设置nvm登录自启动
在`~/.bashrc`，`~/.profile`，`~/.zshrc`文件下添加下面几行代码
```shell
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion
```


检查一下nvm是否安装成功
```shell
command -v nvm
```

## 指定Nodejs版本
到Nodejs官网查看最新的稳定版本`LTS`，我现在是10.15.3 LTS
```shell
nvm install v10.15.3
```
nvm指定版本
```shell
nvm use v10.15.3
nvm alias default v10.15.3
node -v // check一下是否安装成功
```
如果服务器在国内，速度比较慢，可以添加淘宝源
```shell
npm --regitstry=https://regitstry.npm.taobao.org install -g npm
npm -v // 查看版本 查看安装成功
```
如果实在拉取不到一些模块，可以安装`cnpm`
用cnpm替代npm 
```
npm --regitstry=https://regitstry.npm.taobao.org install -g cnpm
cnpm -v
```

## 全局安装常用的工具包
```
npm i pm2 hexo-cli -g
```

## 测试nodejs环境
新建app.js文件，写一个最简单的服务器程序
``` javascript
const http = requrire('http')
http.createServer(function(req, res) {
    res.writeHead(200, {'Content-Type': 'text/plain'})
    res.end('Hello World')
    }).listen(8081) // 国内服务器可能没有开启80和8080端口权限
```
启动服务器脚本
```shell
node app.js
```
检查一下，浏览器访问`http:/{ip address}:8081`

### pm2管理nodejs进程
pm2可以可以后台启动nodejs进程，遇到异常时自动重启，并且记录日志
```
`pm2 start app.js`
`pm2 list` // 列出当前node服务
`pm2 show app` // 产看应用的详细信息
`pm2 log` // 查看实时日志
```

## 安装Nginx
```shell
sudo apt-get update
sudo apt-get install nginx
nginx -v
```

## 配置Ngnix


## DNSPod域名管理
在域名商网站，使用自定义的域名服务器，我这里使用DNSPod
输入DNSPod的2个DNS短地址
```
f1g1ns1.dnspod.net
f1g1ns2.dnspod.net
```
在DNSPod里添加一条A记录，`主机记录: www 记录类型: A 记录值: IP地址`
```
A记录 // 指向IP地址
CNAME // 指向域名，比如指向七牛的资源服务器域名（无论七牛服务器ip怎么变)
MX // 让邮箱能收到邮件
```
这里添加不同的记录，可以使用不同的二级域名，在同一服务器上，部署多个项目

## 搭建git仓库 
在github.com上new一个新的repository，本地上传项目源码
```shell
git init
git commit -m "first commit"
git remote add origin git@github.com:hw-in-github/blog.git
git push -u origin master
```

## 部署项目
### pm2配置文件
在项目里新建一个`ecosystem.json`
```json
{
  "apps": [
    {
      "name": "blog",
      "script": "hexo server -p 8081",
      "env": {
        "COMMON_VARIABLE": "true"
      },
      "env_production": {
        "NODE_ENV": "production"
      }
    }
    "deploy": {
      "production": {
        "user": "root",
        "host": ["107.151.139.139"],
        "port": "26434",
        "ref": "origin/master",
        "repo": "https://github.com/hw-in-github/blog.git",
        "path": "/www/blog/production",
        "ssh_options": "StrictHostKeyChecking=no",
        "env": {
          "NODE_ENV": "production"
        }
      }
    }
  ]
}
````
```
在服务器上新建项目运行的文件夹
```
sudo mkdir /www
cd /www
mkdir blog
sudo chmod 777 blog // pm2通过ssh，需要访问这个文件夹，这里先把权限都打开，不太安全
```
### 执行部署
```shell
pm2 deploy ecosystem.json production setup
```