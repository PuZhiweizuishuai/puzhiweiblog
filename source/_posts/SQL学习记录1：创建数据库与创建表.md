---
title: SQL学习记录1：创建数据库与创建表
date: 2018-05-14 22:22:58
tags: [SQL,数据库]
categories: "数据库"
copyright: true
top: 99
---
<img src="http://7xjjdc.com1.z0.glb.clouddn.com/blog/sql/network-3396348_1280.gif" alt="数据库1">

{% note class_name %}
**本文为SQL学习记录笔记** 
**第一篇记录笔记：创建数据库与创建表**
{% endnote %}
<!-- more --> 

## 创建数据库的基本语法如下所示：

{% codeblock 创建数据库 lang:SQL %}
CREATE DATABASE sqlbizhibihui --创建一个叫sqlbizhibihui的数据库
ON PRIMARY
(
  name = 'sqlbizhibihui', --主数据文件逻辑名称
  filename = 'F:\visual studio 2015\sqlbizhibihui\sqlbizhibihui.mdf', --主数据文件实际保存路径
  size = 5MB, --主数据文件初始大小
  maxsize = 150MB, --最大容量
  filegrowth = 20% --以20%扩容
)
LOG ON --配置日志文件选项
(
  name = 'sqlbizhibihui_log',--日志文件逻辑名称
  filename = 'F:\visual studio 2015\sqlbizhibihui\sqlbizhibihui_log.ldf',--日志文件保存地址
  size = 5MB, --日志文件初始大小
  filegrowth = 5MB--超过默认值后再扩容5mb
)

{% endcodeblock %}

## 创建表的基本语法如下所示：

{% codeblock 创建表 lang:SQL %}
CREATE TABLE table_name --创建一个名为table_name的表
(
field1 data_type1 not null, --field1列名 data_type数据类型 not null 不允许空值 
field2 data_type2 not null, 
field3 data_type3 not null, 
field4 data_type4 not null, 
field5 data_type5 not null
); 
{% endcodeblock %}

## 修改表的基本语法如下所示：

### 1.修改表元素

{% codeblock 修改表元素 lang:SQL %}
ALTER TABLE EMPLOYEE_TEL 
ALTER COLUMN EMP_ID VARCHAR(10)      --修改表EMP_ID数据类型为varchar（10）

--注意SQL server使用ALTER COLUMN,而MySQL和Oracle要使用
ALTER TABLE EMPLOYEE_TEL
MODIFY EMP_ID VARCHAR(10);
{% endcodeblock %}

### 2.添加列

所有的 DBMS 都允许给现有的表增加列，不过对所增加列的数据类型
（以及 NULL 和 DEFAULT 的使用）有所限制。

{% codeblock 添加列 lang:SQL %}
ALTER TABLE Vendors
ADD vend_phone CHAR(20); --给表Vendors添加一列vend_phone,数据类型为Char（20） 
{% endcodeblock %}


### 3.添加自动增加的列

MySQL提供SERIAL方法为表生成唯一值，例如

{% codeblock Mysql添加自动增加的列 lang:SQL %}
CREATE TABLE TEST_INCREMENT
(
    ID SERIAL,
    TEST_NAME VARCHAR(20),
);
{% endcodeblock %}

SQL Server使用IDENTITY类型

{% codeblock SQL Server添加自动增加的列 lang:SQL %}
CREATE TABLE TEST_INCREMENT
(
    ID INT IDENTITY(1,1) NOT NULL,
    TEST_NAME VARCHAR(20),
);
{% endcodeblock %}

然后就可以直接插入数据了

{% codeblock 添加自动增加的列 lang:SQL %}
INSERT INTO TEST_INCREMENT(TEST_NAME)
VALUES ('FRED'),('JOE'),('MIKE'),('TED');
{% endcodeblock %}

### 4.从现有表新建另一个表

SQL Server使用以下代码

{% codeblock lang:SQL %}
SELECT * 
INTO PRODUCTS_TMP     --要复制到的新表名
FROM PRODUCTS_TBL;        --需要复制的表名
{% endcodeblock %}

MySQL和Oracle使用以下代码

{% codeblock lang:SQL %}
CREATE TABLE PRODUCTS_TMP AS--要复制到的新表名
SELECT * FROM PRODUCTS_TBL; --需要复制的表名
{% endcodeblock %}

### 5.删除表
MySQL和Oracle使用以下代码

{% codeblock lang:SQL %}
DROP TABLE TABLE_NAME [ RESTRICT | CASCADE ]
{% endcodeblock %}

如果使用了RESTRICT选项，并且表被视图或约束所引用，DROP就会返回一个错误。
而如果使用CASCADE选项，删除就会成功，而且表引用的约束和视图引用都会被删除

SQL Server使用以下代码

{% codeblock lang:SQL %}
DROP TABLE PRODUCTS_TMP;

SQL Server不支持CASCADE选项，所以必须先删除所有引用关系所有对象，以避免系统中遗留无效对象
{% endcodeblock %}


## 完整性约束

### 1.主键约束
把字段EMP_ID指定为表EMPLOYEE_TBL的主键

{% codeblock lang:SQL %}
CREATE TABLE EMPLOYEE_TBL
(
	EMP_ID		VARCHAR(9)		NOT NULL,
	LAST_NAME	VARCHAR(15)		NOT NULL,
	FIRST_NAME	VARCHAR(15)		NOT NULL,
	MIDDLE_NAME	VARCHAR(15),
	[ADDRESS]	VARCHAR(30)		NOT NULL,
	CITY		VARCHAR(15)		NOT NULL,
	[STATE]		CHAR(2)			NOT NULL,
	ZIP			INTEGER			NOT NULL,
	PHONE		CHAR(10),
	PAGER		CHAR(10)
	PRIMARY KEY (EMP_ID)
);
{% endcodeblock %}

以上为在创建表时设定EMP_ID为主键，还可以在创建完表后为表添加主键
以下命令为为表EMPLOYEE_TBL添加规则n1，设置EMP_ID为主键

{% codeblock lang:SQL %}
alter table EMPLOYEE_TBL
	add constraint n1 primary key(EMP_ID)
{% endcodeblock %}


### 2.唯一性约束
唯一性约束要求表中某个字段的值在每条记录里都是唯一的，这一点和主键类似。
为phone列添加唯一性约束

{% codeblock lang:SQL %}
CREATE TABLE EMPLOYEE_TBL
(
	EMP_ID		VARCHAR(9)		NOT NULL,
	LAST_NAME	VARCHAR(15)		NOT NULL,
	FIRST_NAME	VARCHAR(15)		NOT NULL,
	PHONE		CHAR(10),         UNIQUE,
	PRIMARY KEY (EMP_ID)
);
{% endcodeblock %}


### 3.外键约束
外键是子表里的一个字段，引用父表里的主键，外键约束是确保表于表之间引用完整性的主要机制，一个被定义为外键的字段用于引用另一个表里的主键。
在这个范例中，EMP_ID字段被定义为表EMPLOYEE_PAY_TST的外键，它引用了表EMPLOYEE_TBL里的EMP_ID里的字段，这个外键确定了表EMPLOYEE_PAY_TST里每个EMP_ID都在EMPLOYEE_TBL里有对应的EMP_ID，这被称为父子关系，其中父表是EMPLOYEE_TBL，字表是EMPLOYEE_PAY_TST。

{% codeblock lang:SQL %}
CREATE TABLE EMPLOYEE_PAY_TST
(
	EMP_ID CHAR(9) NOT NULL,
	POSITION VARCHAR(15) NOT NULL,
	DATE_HIRE DATE NULL,
	CONSTRAINT EMP_ID_FK FOREIGN KEY(EMP_ID) REFERENCES EMPLOYEE_TBL(EMP_ID)
);
{% endcodeblock %}

为了在字表里插入一个EMP_ID值，它首先要存在于父表的EMP_ID里，类似的，父表里删除一个EMP_ID，字表里的EMP_ID也必须完全删除
还可以通过以下方式设置外键：

{% codeblock lang:SQL %}
alter table sc                --设置要修改的表名
	add constraint fk_Id      --设置外键名
	foreign key(id)            --关联的列名
	references student(id)       --关联的表名的表中的列名
{% endcodeblock %}

### 4.检查约束
检查约束用于检查输入到特定字段的数据的有效性，限制约束限制对特定输入数据的范围或格式，确保该列获得有效值，避免非法数据类型的产生与扩散。对同一列可以定义多个检查约束。但标识列。ROWGUIDCOL列或数据类型为TIMESTAMP的列不能定义检查约束，因为他们的约束由数据库系统字动添加。 
基本格式

{% codeblock lang:SQL %}
CREATE TABLE 表名
(
	列名 数据类型 其他约束 CONSTRAINT 检查约束名 CHECK (约束表达式)
)
{% endcodeblock %}

例如设置表XSSCORE中列SCORE的值大于0小于100

{% codeblock lang:SQL %}
CREATE TABLE XSSCORE
(
	SCORE DECIMAL(4,1) NULL CHECK(SCORE>=0 AND SCORE<=100)
)
{% endcodeblock %}

### 5.默认值约束
基本格式

{% codeblock lang:SQL %}
CREATE TABLE 表名
(
	列名 数据类型 其他约束 CONSTRAINT 默认约束名 DEFAULT (约束表达式)
)
{% endcodeblock %}

例如，设置表nation默认值为汉

{% codeblock lang:SQL %}
CREATE TABLE XSSCORE
(
	NATION VARCHAR(8) NOT NULL DEFAULT ‘汉’
)
{% endcodeblock %}

还可以通过以下语句添加默认约束

{% codeblock lang:SQL %}
ALTER TABLE 表名
	ADD CONSTRAINT 默认约束名 DEFAULT 默认表达式 FOR 列名
{% endcodeblock %}

例如为表student列nation添加默认值汉

{% codeblock lang:SQL %}
ALTER TABLE Student
	ADD CONSTRAINT DF_Student DEFAULT '汉' FOR nation
{% endcodeblock %}

### 6.默认值
默认值与默认约束类似，但是默认约束与默认值不能同时在某个列上使用
默认值可以绑定到多个不同的列上

{% codeblock lang:SQL %}
go
CREATE DEFAULT DF_NATION AS '汉族'     --创建名为DF_NATION的约束
go
EXEC sp_bindefault DF_NATION ,'STUDENT.NATION'  --将约束绑定到表student中的列nation
go
删除默认值
GO
EXEC sp_unbindefault 'STUDENT.NATION' --解除约束绑定
GO
DROP DEFAULT DF_NATION  --删除默认值对象
GO
{% endcodeblock %}

### 7.规则
设置score取值范围为[0,100]

{% codeblock lang:SQL %}
go
--创建规则，设置规则名为r1_score
create rule rl_score as @score>=0 and @score<=100
go
--绑定规则
exec sp_bindrule rl_score,'sc.score'
go

--查看sc中所有规则
go
sp_help sc
go
{% endcodeblock %}

删除规则

{% codeblock lang:SQL %}
go
exec sp_unbindrule 'sc.score'    --解除绑定
go
drop rule rl_score               --删除规则
go
{% endcodeblock %}


8.删除约束

删除表EMPLOYEES里的EMPlOVEES_PK约束

{% codeblock lang:SQL %}
ALTER TABLE EMPLOYEES 
DROP CONSTRAINT EMPlOVEES_PK
{% endcodeblock %}