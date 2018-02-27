---
author: heh3.org
comments: true
date: 2017-05-08 03:59:42+00:00
layout: post
link: http://heh3.org/?p=99
slug: centos-7-5-%e5%ae%89%e8%a3%85docker-ce
title: CentOS 7.5 安装Docker CE
wordpress_id: 99
categories:
- linux
---

Docker CE is supported on CentOS 7.3 64-bit.






  * 


Set up the repository



    
    <code>sudo yum install -y yum-utils
    
    sudo yum-config-manager \
        --add-repo \
        https://download.docker.com/linux/centos/docker-ce.repo
    
    sudo yum makecache fast
    </code>






  * 


Get Docker CE




Install the latest version of Docker CE on CentOS:



    
    <code>sudo yum -y install docker-ce
    </code>





Start Docker:



    
    <code>sudo systemctl start docker
    </code>






  * 


Test your Docker CE installation




Test your installation:



    
    <code>sudo docker run hello-world
    </code>






  * 


Docker 官方为了简化安装流程，提供了一套安装脚本，CentOS 系统上可以使用这
套脚本安装： 



    
    <code>curl -sSL https://get.docker.com/ | sh
    </code>





执行这个命令后，脚本就会自动的将一切准备工作做好，并且把 Docker 安装在系
统中。
不过，由于伟大的墙的原因，在国内使用这个脚本可能会出现某些下载出现错误的
情况。国内的一些云服务商提供了这个脚本的修改版本，使其使用国内的 Docker
软件源镜像安装，这样就避免了墙的干扰。





  * 


阿里云的安装脚本



    
    <code>curl -sSL http://acs-public-mirror.oss-cn-hangzhou.aliyuncs.com/
    docker-engine/internet | sh -
    </code>






  * 


DaoCloud 的安装脚本



    
    <code>curl -sSL https://get.daocloud.io/docker | sh
    </code>






  * 


添加内核参数




默认配置下，在 CentOS 使用 Docker 可能会碰到下面的这些警告信息：



    
    <code>    WARNING: bridge-nf-call-iptables is disabled
        WARNING: bridge-nf-call-ip6tables is disabled
    </code>





添加内核配置参数以启用这些功能。



    
    <code>$ sudo tee -a /etc/sysctl.conf <<-EOF
    net.bridge.bridge-nf-call-ip6tables = 1
    net.bridge.bridge-nf-call-iptables = 1
    EOF
    </code>





然后重新加载 sysctl.conf 即可



    
    <code>$ sudo sysctl -p
    </code>





如果 docker version 、 docker info 都正常的话，可以运行一个 Nginx 服务
器：



    
    <code>$ docker run -d -p 80:80 --name webserver nginx
    </code>





服务运行后，可以访问 http://localhost，如果看到了 "Welcome to nginx!"




要停止 Nginx 服务器并删除执行下面的命令：



    
    <code>$ docker stop webserver
    $ docker rm webserver
    </code>







