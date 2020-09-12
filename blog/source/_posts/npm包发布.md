---
title: npm包发布
tags:
  - node
  - null
categories:
  - node
  - null
date: 2020-08-27 16:12:59
mp3:
cover:
---
教你怎么发布一个自己的npm包
<!-- more -->

1. 编写测试文件

   ~~~js
   exports.sayHello = function() {
     console.log('hello')
   }
   ~~~

2. 使用`npm init`初始化包描述文件`package.json`

3. 注册包仓库账号

   ~~~shell
   npm adduser
   ~~~

4. 上传包

   ~~~shell
   npm publish .
   ~~~

   上传包的时候一定要用官方的镜像，否则会出错，这里推荐镜像管理工具`nrm`

   ⚠️注意发布的包不要和npm官方仓库的包重名，以及需要验证你的邮箱

5. 下载测试发布的包

6. 管理包权限

   通常你发布的包，只能自己维护，如果其他人也想有权限，可以使用下面的命令进行管理

   ~~~shell
   npm owner ls <package name> 
   npm owner add <user> <package name> 
   npm owner rm <user> <package name>
   ~~~
