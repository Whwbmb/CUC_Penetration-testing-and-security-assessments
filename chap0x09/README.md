# 第四章：渗透测试靶场实战
## 安装 DVWA 靶场
### 1. 创建以下 Dockerfile
首先创建一个Dockerfile文件，并写入如下内容

![docker](./img_9/dockerfile.png)

```
 FROM vulnerables/web-dvwa
 RUN sed -i ‘s/allow_url_fopen = Off/allow_url_fopen = On/’/etc/php/7.0/apache2/php.ini &&
  sed -i ‘s/allow_url_include =Off/allow_url_include = On/’ /etc/php/7.0/apache2/php.ini
```

![dokcer](./img_9/dockerfile0.png)

### 2. 构建 DVWA 镜像及容器
依次执行如下命令：
```
    docker pull vulnerables/web-dvwa
    docker build --tag my_dvwa .
    docker run --rm -it -d -p 4090:80 my_dvwa
```
* 拉取容器文件
  
![docker](./img_9/创建容器.png)

* 创建容器
  
![docker](./img_9/创建容器1.png)

* 启动容器
  
![docker](./img_9/创建容器2.png)

进入浏览器查看页面：
账号为：admin
密码为：password

![docker](./img_9/登录1.png)

登录后页面如下：

![docker](./img_9/登录2.png)

按要求创建数据库：

![docker](./img_9/登录3.png)

## 实验⼀：DVWA - Vulnerability: File Upload - Low
### 步骤⼀：
1. 尝试上传任意⽂件，使⽤其正常功能
经验证，起上传功能正常，且能够上传多种类型文件
2. 尝试上传⼀个phpinfo⽂件，验证上传的phpinfo⽂件是否可执⾏。
文件代码如下：
```
<?php
 // Show all information, defaults to INFO_ALL
 phpinfo();
 ?>
```
将其提交到网页后打开提示的网页，发现显示了所有的信息，证明文件可执行：

![docker](./img_9/exp1/low1.png)

3. 尝试上传⼀个 Web Shell ⽂件
进入如下链接，将代码复制并创建一个新的php文件
   https://github.com/artyuum/simple-php-web-shell/blob/master/index.php
   同理上传并打开

   ![docker](./img_9/exp1/low2.png)

 4. 验证是否可以执⾏任意命令
输入`ls`进行验证
验证成功

![docker](./img_9/exp1/low3.png)

再输入`cat`进行验证
验证成功

![docker](./img_9/exp1/low4.png)

### 步骤二：分析漏洞原因
由图可知没有对文件类型进行限制

![docker](./img_9/exp1/low5.png)


### 步骤三：使⽤ Burpsuite 抓包查看通信流量
通过抓包并分析包的内容也可以发现没有对文件类型进行限制

![docker](./img_9/exp1/low6.png)

## 实验二：：DVWA - Vulnerability: File Upload - Medium
### 步骤⼀
首先提升安全难度到Medium

![docker](./img_9/exp2/m1.png)

1. 尝试上传 Web Shell，发现上传失败。
   
![docker](./img_9/exp2/m3.png)

2. 查看源码，查找过滤规则。发现对于⽂件类型做出了限制，要求必须是"image/jpeg" 或者 "image/png"

![docker](./img_9/exp2/m2.png)

3. 在 Burpsuit 的 Repeater 中，将请求头中的 Content-Type: application/x-php
修改为 Content-Type: image/png，再次尝试发送上传 Web Shell 的请求。
先进行抓包，获取请求包，发现其中type是Content-Type: application/x-php

![docker](./img_9/exp2/m4.png)

将请求包进行重发，并修改其中的type部分为Content-Type: image/png

![docker](./img_9/exp2/m5.png)

根据回应结果显示，上传成功

![docker](./img_9/exp2/m6.png)

![docker](./img_9/exp2/m7.png)

## （扩展）实验三：DVWA - Vulnerability: File Upload High

#### 首先修改难度为high

![](./img_9/exp3/h1.png)

1. 尝试上传 Web Shell，发现上传失败。
   
![](./img_9/exp3/h2.png)

2. 查看源码，查找过滤规则。发现对于⽂件后缀做出了限制，要求必须是jpg、jpeg 或者 png，⽂件⼤⼩必须⼩于100000字节
  
![](./img_9/exp3/h3.png)

3. 代码通过 getimagesize( $uploaded_tmp ) 函数，取了图⽚⽂件的⼤⼩，也就是说，上传的⽂件必须符合图⽚的⽂件头信息。因此，需要将webshell防在⼀个可以正常上传的图⽚的后⾯，才能通过校验。
先上传一个图图片文件，看是否符合要求，能够上传成功

![](./img_9/exp3/h4.png)

经上一步验证可以上传成功后，在图片文件的最后面直接添加php代码

![](./img_9/exp3/h5.png)

然后将添加了php文件后的图片文件再次上传

![](./img_9/exp3/h6.png)

4. 当上传成功后，访问上传⽂件路径，发现显⽰了⼀张图⽚，上传的webshell并
没有被执⾏，为了将图片中的php文件执行，需要修改难度为low，然后进行文件包含攻击

![](./img_9/exp3/h8.png)

在low难度的文件包含攻击中，通过点击示例的php文件可以发现在网址的`page=`后面即为打开的目录，因此可以将文件上传攻击中上传成功时获得的目录复制到`page=`后面来打开并执行php代码

![](./img_9/exp3/h9.png)

进行完上步操作后，页面显示如下下，php代码的效果出现，通过输入`ls`、`pwd`测试效果发现可用

![](./img_9/exp3/h7.png)

![](./img_9/exp3/h10.png)

## 遇到的问题
拓展实验中的high难度下脚本无法远程执行，可以在high难度下进行文件的上传然后将难度调调为low后在文件包含攻击中执行

## 参考文献 
https://blog.csdn.net/weixin_47598409/article/details/109063778
https://www.cnblogs.com/renletao/p/13740383
https://blog.csdn.net/weixin_43726831/article/details/102534850