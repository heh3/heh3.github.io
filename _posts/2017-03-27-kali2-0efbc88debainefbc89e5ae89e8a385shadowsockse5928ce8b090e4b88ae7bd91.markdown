---
author: heh3.org
comments: true
date: 2017-03-27 10:54:53+00:00
layout: post
link: http://heh3.org/?p=80
slug: kali2-0%ef%bc%88debain%ef%bc%89%e5%ae%89%e8%a3%85shadowsocks%e5%92%8c%e8%b0%90%e4%b8%8a%e7%bd%91
title: Kali2.0（debain）安装shadowsocks和谐上网
wordpress_id: 80
---

### Ubuntu/debain安装shadowsocks



和谐上网的前提是你得有一个shadowsocks服务器


用PIP安装很简单，







 	
  1. 




    
    <code><span style="color: black; font-family: Consolas;">sudo apt<span style="color: #666600;">-<span style="color: #000088;">get<span style="color: black;"> update</span>
    									</span></span></span></code>





 	
  2. 




    
    <code><span style="color: black; font-family: Consolas;">sudo apt<span style="color: #666600;">-<span style="color: #000088;">get<span style="color: black;"> install python<span style="color: #666600;">-<span style="color: black;">pip</span>
    											</span></span></span></span></span></code>





 	
  3. 




    
    <code><span style="color: black; font-family: Consolas;">sudo apt<span style="color: #666600;">-<span style="color: #000088;">get<span style="color: black;"> install python<span style="color: #666600;">-<span style="color: black;">setuptools m2crypto</span>
    											</span></span></span></span></span></code>








接着安装shadowsocks





pip install shadowsocks





如果是ubuntu16.04 直接 (16.04 里可以直接用apt 而不用 apt-get 这是一项改进）





sudo apt install shadowsocks





当然你在安装时候肯定有提示需要安装一些依赖比如python-setuptools m2crypto ，依照提示安装然后再安装就好。也可以网上搜索有很多教程的。






### 启动shadowsocks





安装好后，在本地我们要用到sslocal ，终端输入sslocal --help 可以查看帮助，像这样





![](https://heh3.org/wp-content/uploads/2017/03/032717_1027_Kali20debai1.png)





通过帮助提示我们知道各个参数怎么配置，比如 sslocal -c 后面加上我们的json配置文件，或者像下面这样直接命令参数写上运行。





比如 sslocal -s 11.22.33.44 -p 50003 -k "123456" -l 1080 -t 600 -m aes-256-cfb





-s表示服务IP, -p指的是服务端的端口，-l是本地端口默认是1080, -k 是密码（要加""）, -t超时默认300,-m是加密方法默认aes-256-cfb，





**为了方便我推荐直接用sslcoal -c 配置文件路径 这样的方式，简单好用。**





我们可以在/home/mudao/ 下新建个文件shadowsocks.json  (mudao是我在我电脑上的用户名，这里路径你自己看你的)。内容是这样：





    
    <code><span style="font-family: Consolas;"><span style="color: #666600;">{</span>
    					</span></code>



    
    <code><span style="color: #008800; font-family: Consolas;">"server"<span style="color: #666600;">:<span style="color: #008800;">"11.22.33.44"<span style="color: #666600;">,</span>
    							</span></span></span></code>



    
    <code><span style="color: #008800; font-family: Consolas;">"server_port"<span style="color: #666600;">:<span style="color: #006666;">50003<span style="color: #666600;">,</span>
    							</span></span></span></code>



    
    <code><span style="color: #008800; font-family: Consolas;">"local_port"<span style="color: #666600;">:<span style="color: #006666;">1080<span style="color: #666600;">,</span>
    							</span></span></span></code>



    
    <code><span style="color: #008800; font-family: Consolas;">"password"<span style="color: #666600;">:<span style="color: #008800;">"123456"<span style="color: #666600;">,</span>
    							</span></span></span></code>



    
    <code><span style="color: #008800; font-family: Consolas;">"timeout"<span style="color: #666600;">:<span style="color: #006666;">600<span style="color: #666600;">,</span>
    							</span></span></span></code>



    
    <code><span style="color: #008800; font-family: Consolas;">"method"<span style="color: #666600;">:<span style="color: #008800;">"aes-256-cfb"</span>
    						</span></span></code>



    
    <code><span style="font-family: Consolas;"><span style="color: #666600;">}</span>
    					</span></code>




server  你服务端的IP
servier_port  你服务端的端口
local_port  本地端口，一般默认1080
passwd  ss服务端设置的密码
timeout  超时设置 和服务端一样
method  加密方法 和服务端一样





确定上面的配置文件没有问题，然后我们就可以在终端输入 sslocal -c /home/mudao/shadowsocks.json 回车运行。如果没有问题的话，下面会是这样...





![](https://heh3.org/wp-content/uploads/2017/03/032717_1027_Kali20debai2.png)（如果继续请不要关闭这个终端）





正常情况下其实都可以用了，但是我们执行之后会发现，在kali 2.0 上并非如此，而是会出现undefined symbol: EVP_CIPHER_CTX_cleanup错误。





    
    <code><span style="color: #333333; font-family: Consolas; font-size: 10pt;">INFO: loading config from ss.json 
    </span></code>



    
    <code><span style="color: #333333; font-family: Consolas; font-size: 10pt;">2016-12-14 22:47:50 INFO loading libcrypto from libcrypto.so.1.1 
    </span></code>



    
    <code><span style="color: #333333; font-family: Consolas; font-size: 10pt;">Traceback (most recent call last): 
    </span></code>



    
    <code><span style="color: #333333; font-family: Consolas; font-size: 10pt;">File "/usr/local/bin/sslocal", line 11, in 
    </span></code>



    
    <code><span style="color: #333333; font-family: Consolas; font-size: 10pt;">sys.exit(main()) 
    </span></code>



    
    <code><span style="color: #333333; font-family: Consolas; font-size: 10pt;">File "/usr/local/lib/python2.7/dist-packages/shadowsocks/local.py", line 39, in main 
    </span></code>



    
    <code><span style="color: #333333; font-family: Consolas; font-size: 10pt;">config = shell.get_config(True) 
    </span></code>



    
    <code><span style="color: #333333; font-family: Consolas; font-size: 10pt;">File "/usr/local/lib/python2.7/dist-packages/shadowsocks/shell.py", line 262, in get_config 
    </span></code>



    
    <code><span style="color: #333333; font-family: Consolas; font-size: 10pt;">check_config(config, is_local) 
    </span></code>



    
    <code><span style="color: #333333; font-family: Consolas; font-size: 10pt;">File "/usr/local/lib/python2.7/dist-packages/shadowsocks/shell.py", line 124, in check_config 
    </span></code>



    
    <code><span style="color: #333333; font-family: Consolas; font-size: 10pt;">encrypt.try_cipher(config['password'], config['method']) 
    </span></code>



    
    <code><span style="color: #333333; font-family: Consolas; font-size: 10pt;">File "/usr/local/lib/python2.7/dist-packages/shadowsocks/encrypt.py", line 44, in try_cipher 
    </span></code>



    
    <code><span style="color: #333333; font-family: Consolas; font-size: 10pt;">Encryptor(key, method) 
    </span></code>



    
    <code><span style="color: #333333; font-family: Consolas; font-size: 10pt;">File "/usr/local/lib/python2.7/dist-packages/shadowsocks/encrypt.py", line 83, in init 
    </span></code>



    
    <code><span style="color: #333333; font-family: Consolas; font-size: 10pt;">random_string(self._method_info[1])) 
    </span></code>



    
    <code><span style="color: #333333; font-family: Consolas; font-size: 10pt;">File "/usr/local/lib/python2.7/dist-packages/shadowsocks/encrypt.py", line 109, in get_cipher 
    </span></code>



    
    <code><span style="color: #333333; font-family: Consolas; font-size: 10pt;">return m[2](method, key, iv, op) 
    </span></code>



    
    <code><span style="color: #333333; font-family: Consolas; font-size: 10pt;">File "/usr/local/lib/python2.7/dist-packages/shadowsocks/crypto/openssl.py", line 76, in init 
    </span></code>



    
    <code><span style="color: #333333; font-family: Consolas; font-size: 10pt;">load_openssl() 
    </span></code>



    
    <code><span style="color: #333333; font-family: Consolas; font-size: 10pt;">File "/usr/local/lib/python2.7/dist-packages/shadowsocks/crypto/openssl.py", line 52, in load_openssl 
    </span></code>



    
    <code><span style="color: #333333; font-family: Consolas; font-size: 10pt;">libcrypto.EVP_CIPHER_CTX_cleanup.argtypes = (c_void_p,) 
    </span></code>



    
    <code><span style="color: #333333; font-family: Consolas; font-size: 10pt;">File "/usr/lib/python2.7/ctypes/init.py", line 375, in getattr 
    </span></code>



    
    <code><span style="color: #333333; font-family: Consolas; font-size: 10pt;">func = self.getitem(name) 
    </span></code>



    
    <code><span style="color: #333333; font-family: Consolas; font-size: 10pt;">File "/usr/lib/python2.7/ctypes/init.py", line 380, in getitem 
    </span></code>



    
    <code><span style="color: #333333; font-family: Consolas; font-size: 10pt;">func = self._FuncPtr((name_or_ordinal, self)) 
    </span></code>



    
    <code><span style="color: #333333; font-family: Consolas; font-size: 10pt;">AttributeError: /usr/lib/x86_64-Linux-gnu/libcrypto.so.1.1: undefined symbol: EVP_CIPHER_CTX_cleanup
    </span></code>




**这是由于openssl升级到1.1.0以上版本后，废弃了EVP_CIPHER-CTX_cleanup函数导致的，如官网所说：**





    
    <code><span style="color: #333333; font-family: Consolas; font-size: 10pt;">EVP_CIPHER_CTX was made opaque in OpenSSL 1.1.0. As a result, EVP_CIPHER_CTX_reset() appeared and EVP_CIPHER_CTX_cleanup() disappeared. 
    </span></code>




变为





    
    <code><span style="color: #333333; font-family: Consolas; font-size: 10pt;">EVP_CIPHER_CTX_init() remains as an alias for EVP_CIPHER_CTX_reset().
    </span></code>




**修改方法：
**




**用vim打开文件：vim /usr/local/lib/python2.7/dist-packages/shadowsocks/crypto/openssl.py (该路径请根据自己的系统情况自行修改，如果不知道该文件在哪里的话，可以使用find命令查找文件位置)跳转到52行（shadowsocks2.8.2版本，其他版本搜索一下cleanup）进入编辑模式将第52行libcrypto.EVP_CIPHER_CTX_cleanup.argtypes = (c_void_p,)
**




**改为libcrypto.EVP_CIPHER_CTX_reset.argtypes = (c_void_p,)再次搜索cleanup（全文件共2处，此处位于111行），将libcrypto.EVP_CIPHER_CTX_cleanup(self._ctx)
**




**改为libcrypto.EVP_CIPHER_CTX_reset(self._ctx)保存并退出启动shadowsocks服务：service shadowsocks start 或 sslocal -c ss配置文件目录问题解决**





以上就是Kali2.0 update到最新版本后安装shadowsocks服务报错问题的全文介绍,希望对您学习和使用linux系统开发有所帮助.





Refer:





[http://blog.csdn.net/blackfrog_unique/article/details/60320737](http://blog.csdn.net/blackfrog_unique/article/details/60320737)





[https://aitanlu.com/ubuntu-shadowsocks-ke-hu-duan-pei-zhi.html](https://aitanlu.com/ubuntu-shadowsocks-ke-hu-duan-pei-zhi.html)

