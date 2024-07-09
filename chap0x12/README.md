# 第六章：安全评估实战
## 实验⼀：OWASP Juice Shop 实战 Broken Access Control 
### 步骤⼀：查看其他⽤户的购物篮
先尝试使⽤正常的购物篮功能,并捕获相关⽹络流量

![basket](./img_12/exp1/查看他人购物栏3.png)

**查看⽹络请求，发现其中请求头的`GET`变量中的`basket/`后的数字可能就是用户的ID，通过验证可知猜测正确**
  
![basket](./img_12/exp1/查看他人购物栏.png)

**返回juice shop页面发现已经完成了该题目**
  
![basket](./img_12/exp1/查看他人购物栏2.png)

### 步骤⼆：以其他⽤户⾝份发布产品评论

**通过点击任意一个商品，进入评论发布页面发送一个评论并抓包，查看刚刚发送的数据**

![basket](./img_12/exp1/发布评论.png)

**通过观察上述的包可以发现，发送的评论内容在`message`当中，而发送者在`author`中，所以，只要知道了其他人的邮箱，就能够以他人的名义发送评论到特定商品的评论区页面中，由于之前已经知道了几个人的邮箱，可以用在这里进行测试，比如这里使用bender的账号发送"look at me now!"到苹果汁的评论区中**

![basket](./img_12/exp1/发布评论4.png)

**返回网页后发现已经完成了该题目**

![basket](./img_12/exp1/发布评论5.png)

**进一步验证结果，进入苹果汁的评论区中查看，发现发送的内容确实存在，且是以bender的账号发送的评论**

![basket](./img_12/exp1/发布评论6.png)


### 步骤三：以其他⽤户的名义发表⼀些反馈

**首先进入反馈界面，在网页的左上角点击目录按钮后联系目录下的第一行就是feedback**
**在进入了反馈页面后，按要求输入反馈信息进行测试，并抓取发送的包**

![basket](./img_12/exp1/反馈.png)

![basket](./img_12/exp1/反馈1.png)

**通过观察抓到的包可以发现，`comment`中的内容就是之前在网页上输入的信息，而`UserId`则是用户的ID号，所以进行猜测，只要修改`comment`中的内容和`UserId`就能够以其他人的名义发表反馈，修改部分内容(两个变量中的值都可以任意修改，但ID要有与之对应的值才能够发表成功)后进行测试，根据响应的结果可知反馈成功**

![basket](./img_12/exp1/反馈2.png)

**回到网页后显示完成了该题，进一步确定之前猜测的正确性**

![basket](./img_12/exp1/反馈3.png)

### 步骤四：将其他产品放⼊另⼀个⽤户的购物篮

**先使用自己的账号将产品添加到购物篮中，同时进行抓包，分析包中信息，寻找突破口**

![basket](./img_12/exp1/给他人添加购物车.png)

![basket](./img_12/exp1/给他人添加购物车1.png)

**通过POST包的内容可知，其中最后的几个变量中`ProductId`指的是产品号，`BasketId`指的是添加进入的购物篮号，不同的号为不同用户所有， `quantity`指的是添加进的产品数量**
**修改购物篮号进行尝试，发现包发送失败，报错显示非法购物蓝号**

![basket](./img_12/exp1/给他人添加购物车2.png)

**通过查询和尝试发现需要将原来的`BasketId`同时包含在里面，且必须在修改的`BasketId`之前才能够用发送成功**

![basket](./img_12/exp1/给他人添加购物车3.png)

**返回网页，发现成功完成添加到别人的购物篮中**

![basket](./img_12/exp1/给他人添加购物车5.png)

## 实验⼆：订购圣诞特别惊喜礼盒！（Christmas Special）

### 步骤⼀：尝试发现查找商品信息的接口

**在主页的搜索框随便进行商品搜索，通过抓包，发现查询商品信息的接⼜是 `/rest/products/search?q=`**

![get](./img_12/exp2/search.png)

### 步骤⼆：尝试使⽤ sqlmap 进⾏注⼊

**1. 在进行sqlmap扫描前需要时用到之前抓取到的包。因此，现将之前的查询请求包保存为一个123.txt文件**

![get](./img_12/exp2/123.png)

**2.  使⽤ `sqlmap` 默认参数对⽬标进⾏扫描，发现参数`q`存在`SQL`注⼊**
```
sqlmap -r 123.txt --level 2 -p "q"
```

![get](./img_12/exp2/sqlmap.png)

**3. 进⼀步使⽤ sqlmap 搜集更多结果,尝试注⼊数据库名，发现结果中提示可以直接注⼊表**
```
sqlmap -r 123.txt --level 2 -p "q" --dbms='sqlite' --dbs
```

![get](./img_12/exp2/sqlmap2.png)

**4. 尝试注⼊表名，恢复出来20张表的表名**
```
sqlmap -r 123.txt --level 2 -p "q" --dbms='sqlite' --tables --threads 5
```

![get](./img_12/exp2/sqlmap3.png)

**5. 尝试恢复 Products 表中信息:**
**尝试恢复Products 表中的列名**
```
sqlmap -r 123.txt --level 2 -p "q" --dbms='sqlite' -T Products --columns --no-cast --threads 5
```

**发现列名id和name可利用**

 
**尝试恢复Products表中的指定列，发现⽬标商品！其ID号为10**
```
sqlmap -r 123.txt --level 2 -p "q" --dbms='sqlite' -T Products -C id,name --dump --no-cast --threads 10
```

![get](./img_12/exp2/sqlmap4.png)

### 步骤三：将圣诞特别惊喜礼盒加⼊购物车
**使用之前为其他用户添加商品到购物篮的包将自己的购物篮中添加ID为10的商品，即添加圣诞特别惊喜礼盒到自己的购物篮,根据响应中的success可知添加成功**

![get](./img_12/exp2/Christmas.png)

**回到页面中查看自己的购物篮也能够发现已经添加了目标商品**

![get](./img_12/exp2/Christmas2.png)

**按照要求随便填写地址信息，进行checkout，下单后，页面显示题目完成**

![get](./img_12/exp2/Christmas3.png)



## 实验三：OWASP Juice Shop 实战 - ⾃由扩展
**访问得分榜（score-board）**

![get](./img_12/exp3/score-board.png)

**选择Extra Language这道五星题**

![get](./img_12/exp3/choose_question.png)

**首先在主页面点击语言按钮进行语言切换**

![get](./img_12/exp3/language.png)

**进行抓包，获取请求,同时进行观察，发现`assets/il8n/zh_CN.json`是请求的结构,猜测只要找到目标的json文件替换zh_CN.json文件并成功发送请求即可完成题目**

![get](./img_12/exp3/language1.png)

**联系之前查询别人的购物篮时使用到的结构，进行尝试，发现可以通用到language中，可以查询到所有可用的的language，虽然需要找到的那个json文件不在其中，但可以根据这些json文件名称结构推测目标json文件的名称结构——由两部分组成,可写为以下形式：** ***语言简写_地区简写.json***

![get](./img_12/exp3/detectlanguages.png)

**因此，可以在intruder中设置两个变量进行遍历**

![get](./img_12/exp3/attack.png)

**分别设置其中的参数为所有的小写字母和所有的大写字母**
![get](./img_12/exp3/attack1.png)

![get](./img_12/exp3/attack2.png)

**进行攻击，暴力遍历，按响应的长度排序后可以快速检索出特殊的响应，发现在组合`tth_AA.json`作为文件名称时语言切换成功，猜测`tth_AA.json`就是目标文件**

![get](./img_12/exp3/language2.png)

**将`tth_AA.json`替换原来的`zh_CN.json`文件，发送请求，请求成功**

![get](./img_12/exp3/language3.png)

**回到网页，发现题目完成**

![get](./img_12/exp3/language4.png)

**进入得分板确认题目确实完成**

![get](./img_12/exp3/score-board2.png)

## 遇到的问题
1. 在进行为其他用户购物篮添加商品时请求包发送失败，发现包中`BasketId`报错，通过网络搜索了解到需要将原来的`BasketId`同时包含在里面，且必须在修改的`BasketId`之前才能够用发送成功
2. Extra language这道题目的语言目录中没有要找的那个json文件名，搜索后发现该目录只会显示可用的语言，因此需要根据语言名称结构推测目标json文件的名称
## 参考文献 
1. [【THM】OWASP Juice Shop-练习 - Hekeatsll - 博客园](https://www.cnblogs.com/Hekeats-L/p/16974832.html)

2. [靶场Juice-Shop学习](https://blog.csdn.net/qq_36531487/article/details/113863816)