# 第五章：安全评估基础
## Juice Shop 安装
**1. 执行如下的代码，创建docker容器，进行实验**

```bash
docker run --rm -p 127.0.0.1:3000:3000 bkimminich/juice-shop
```
**2. 执行结果如下图所示，继续进入浏览器的3000端口进入juice shop界面**

![create](./img_11/构建docker.png)

**商店界面如图所示：**

![create](./img_11/商店界面.png)

## 实验⼀：对 Juice Shop 进⾏信息收集
#### 问题 #1：管理员的电⼦邮件地址是什么？

**在商店界面中，点击商品，可以看到用户进行的评价等信息，而在用户进行评价时，其邮箱也会出现在用户名中**

![create](./img_11/exp1/admin.png)

**可知管理员admin的邮箱为`admin@juice-sh.op`**

#### 问题 #2：搜索使⽤了什么参数？

**1. 通过点击右上角的搜索按钮可以进行搜索**

![create](./img_11/exp1/search.png)

**2. 通过搜索随意地内容可以在网址处看到搜索用路径为`/search?q=...`，所以，可知其搜索用的参数为`q`**

![create](./img_11/exp1/search2.png)

#### 问题 #3：Jim 在他的评论中提到了什么电视节⽬？

**1. 首先查找所有的jim的评论**
   
![create](./img_11/exp1/电视节目1.png)

![create](./img_11/exp1/电视节目.png)

![create](./img_11/exp1/电视节目2.png)

**2. 依次分析评论中的信息，第一个中出现了"replicator"这个单词，翻译后发现是"复制器"的意思**


**3. 继续分析其他的评论，得到了"Bones' tricorder"这个关键词汇，进行搜索后发现和一部电影相关，电影为《star trek》**

![create](./img_11/exp1/电视节目0.0.png)


**4. 继续分析，对第三个评论分析，评论中提到了星际舰队，再结合之前的信息，可以合理推测jim所看的节目就是《star trek》（《星际迷航》）**

![create](./img_11/exp1/电视节目2.2.png)

**5. 进行验证，将猜测的结果和关键词同时进行搜索，验证成立，更进一步确定答案为《star trek》（《星际迷航》）**

![create](./img_11/exp1/电视节目3.png)

![create](./img_11/exp1/电视节目1.1.png)

![create](./img_11/exp1/电视节目1.2.png)

## 实验⼆：对 Juice Shop 进⾏注⼊
#### 问题 #1：登录管理员帐户！

**进入的登录界面进行sql注入，由于之前已经知道了管理员的邮箱账号，所以可以直接进行注入攻击,这里使用了两种实现登录管理员账号的方式：**

**1. 使用网页的登录界面直接"填字谜"来实现登录,通过几次尝试发现在邮箱栏输入`admin@juice-sh.op';--`实现注入攻击，其中，`'`实现的是封闭前面的内容，标注登录的是哪个账号，而后面的`--`注释掉了后面的所有内容，直接实现针对某一个账号的登录**

![injection](./img_11/exp2/admin000.png)

![injection](./img_11/exp2/admin1.png)


**2. 使用Burp suit抓取登录的请求后修改实现登录，这种方式和直接使用网页区别不大，只是修改了请求包的内容**
   
![injection](./img_11/exp2/admin0.png)

**3. 此外，使用`' or 1=1;--`也可以直接登录admin账号，因为这样登录的默认账号为ID为0的用户，即管理员**

#### 问题 #2：登录 Bender 帐户！
**方法同登录admin的账号一样，同样的操作即可实现效果**
**1. 首先找到bender的邮箱以进行登录，通过分析其评论，发现其邮箱为`bender@juice-sh.op`**

![injection](./img_11/exp2/bender.png)

**2. 执行和admin登录同样的操作即可,只用修改邮箱名称**

![injection](./img_11/exp2/bender000.png)

![injection](./img_11/exp2/bender0000.png)


**3. 当然也可以通过Burp suit抓包实现**

![injection](./img_11/exp2/bender2.png)

![injection](./img_11/exp2/bender1.png)

**在进行实验2时由于输入错了sql语句，还出现了一个小插曲，即使sql语句没有实现效果，但还是解决了一个"彩蛋"挑战**

![injection](./img_11/exp2/额外1.png)

## 遇到的问题
不理解为什么`' or 1=1;--`也可以登录管理员账号，通过参考资料的解释和网络搜索明白其原因
## 参考文献 
[【THM】OWASP Juice Shop-练习 - Hekeatsll - 博客园](https://www.cnblogs.com/Hekeats-L/p/16974832.html)