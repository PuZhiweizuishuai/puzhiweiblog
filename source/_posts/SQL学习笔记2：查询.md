---
title: SQL学习笔记2：查询
date: 2018-05-15 12:49:39
tags: [SQL,数据库]
categories: "数据库"
copyright: true
top: 98
---
{% note primary %}
**本文为SQL学习记录笔记** 
**第二篇记录笔记：查询**
**包含SELECT语句，简单查询，连接查询及嵌套查询**
{% endnote %}

<!-- more --> 

## SELECT语句基本语法

SELECT语句在任何一种SQL语言中，都是使用频率最高的语句，它具有强大的查询功能，有的用户甚至只需要熟练掌握SELECT语句的一部分，就可以轻松地利用数据库来完成自己的工作。可以说，SELECT语句是SQL语言的灵魂。SELECT语句的作用是让数据库服务器根据客户端的要求搜寻出用户所需要的信息资料，并按用户规定的格式进行整理后返回给客户端。
SELECT语句的基本语法格式如下。
{% codeblock lang:SQL %}
SELECT select_list
[INTO new_table]
FROM table_source
[WHERE search_condition]
[GROUP BY group_by_expression]
[HAVING search_condition]
[ORDER BY order_expression[ASC|DESC]]
{% endcodeblock %}

其中，参数说明如下。
{% codeblock lang:SQL %}
select_list：--指明要查询的选择列表。列表可以包括若干个列名或表达式，列名或表达式之间用逗号隔开，用来指示应该返回哪些数据。表达式可以是列名、函数或常数的列表。
INTO new_table：--指定用查询的结果创建一个新表。new_table为新表名称。 
FROM table_source：--指定所查询的表或视图的名称。
WHERE search_condition：--指明查询所要满足的条件。
GROUP BY group_by_expression：--根据指定列中的值对结果集进行分组。
HAVING search_condition：--对用FROM、WHERE或GROUP BY子句创建的中间结果集进行行的筛选。它通常与GROUP BY子句一起使用。
ORDER BY order_expression [ ASC |DESC]：--对查询结果集中的行重新排序。ASC和DESC关键字分别用于指定按升序或降序排序。如果省略ASC或DESC，则系统默认为升序。
{% endcodeblock %}

例：如查询学生基本信息，则可用

{% codeblock lang:SQL %}
SELECT *          --使用通配符*查询所有列
FROM student      --查询数据来自student表
ORDER BY id;      --按id数值以升序排列，如需降序则需要输入参数DESC
{% endcodeblock %}

## 限制结果

SELECT 语句返回所有匹配的行。但是，如果你不希望每个值每次都出现，该怎么办呢？


办法就是使用 DISTINCT 关键字，顾名思义，它指示数据库只返回不同的值

例如
{% codeblock lang:SQL %}
SELECT DISTINCT courseid       --查询课程，告诉数据库只返回具有唯一性的行数
FROM course; 
{% endcodeblock %}

### 限制行数
在 SQL Server 和 Access 中使用 SELECT 时，可以使用 TOP 关键字来限制
最多返回多少行，如下所示：

{% codeblock lang:SQL %}
SELECT TOP 5 id        --只检索前五行数据
FROM students; 
{% endcodeblock %}


如果你使用 Oracle，需要基于 ROWNUM（行计数器）来计算行，像这样：
{% codeblock lang:SQL %}
SELECT id 
FROM students 
WHERE ROWNUM <=5; 
{% endcodeblock %}



如果你使用 MySQL、MariaDB、PostgreSQL 或者 SQLite，需要使用 LIMIT子句，像这样：

{% codeblock lang:SQL %}
SELECT id
FROM students 
LIMIT 5;             --返回五行数据
{% endcodeblock %}

LIMIT 5指示MySQL等 DBMS 返回不超过 5 行的数据。为了得到后面的 5 行数据，需要指定从哪儿开始以及检索的行数，像这样：
{% codeblock lang:SQL %}
SELECT id
FROM students 
LIMIT 5 OFFSET 5;   --返回五行数据，并从第五行开始
{% endcodeblock %}

## 对结果排序

使用ORDER BY子句

### 单个列排序

{% codeblock lang:SQL %}
SELECT *          --使用通配符*查询所有列
FROM student      --查询数据来自student表
ORDER BY id;      --按id数值以升序排列，如需降序则需要输入参数DESC
{% endcodeblock %}

如

{% codeblock lang:SQL %}
SELECT *          --使用通配符*查询所有列
FROM student      --查询数据来自student表
ORDER BY id DESC;      --按id数值以降序排列
{% endcodeblock %}

**注意：ORDER BY 子句的位置 在指定一条 ORDER BY 子句时，应该保证它是 SELECT 语句中最后一条子句。如果它不是最后的子句，将会出现错误消息。**


### 按多个列排序
要按多个列排序，简单指定列名，列名之间用逗号分开即可（就像选择多个列时那样）

{% codeblock lang:SQL %}
SELECT id, name, score
FROM students
ORDER BY score,name;
{% endcodeblock %}


## 过滤数据

### 使用WHERE子句

例如

查询所有女生的信息

{% codeblock lang:SQL %}
SELECT *
FROM student
WHERE sex='女';        --查询所要满足的条件即都为女生
{% endcodeblock %}


求女生人数
{% codeblock lang:SQL %}
SELECT COUNT(*) AS '女生人数' 
FROM student WHERE sex='女'  --使用COUNT函数返回满足条件列的数目，
--并给改列命名为女生人数
{% endcodeblock %}




**注意：WHERE 子句的位置 在同时使用 ORDER BY 和 WHERE 子句时，应该让 ORDER BY 位于WHERE 之后，否则将会产生错误**


查询不是女生的人
{% codeblock lang:SQL %}
SELECT id, name,sex 
FROM student
WHERE sex <> '女'; 
{% endcodeblock %}


**注意：是!=还是<>？!=和<>通常可以互换。但是，并非所有 DBMS 都支持这两种不等于操作符。例如，Microsoft Access 支持<>而不支持!=。如果有疑问，请参阅相应的 DBMS 文档。**


### IN 操作符

IN 操作符用来指定条件范围，范围中的每个条件都可以进行匹配。IN 取一组由逗号分隔、括在圆括号中的合法值。

查询选择了20001和20002两门课程的学生
{% codeblock lang:SQL %}
SELECT name
FROM students
WHERE couses IN ( 20001, 20002 ) ;
{% endcodeblock %}

为什么要使用 IN 操作符？其优点如下。
1.在有很多合法选项时，IN 操作符的语法更清楚，更直观。
2.在与其他 AND 和 OR 操作符组合使用 IN 时，求值顺序更容易管理。
3.IN 操作符一般比一组 OR 操作符执行得更快（在上面这个合法选项很少的例子中，你看不出性能差异）。
4.IN 的最大优点是可以包含其他 SELECT 语句，能够更动态地建立
WHERE 子句。


### NOT 操作符

WHERE 子句中的 NOT 操作符有且只有一个功能，那就是否定其后所跟的任何条件。因为 NOT 从不单独使用（它总是与其他操作符一起使用），所以它的语法与其他操作符有所不同。NOT 关键字可以用在要过滤的列前，而不仅是在其后。



### LIKE 操作符

LIKE关键字用于查询与指定的某些字符串表达式模糊匹配的数据行。LIKE后的表达式被定义为字符串，必须用半角单引号括起来，字符串中可以使用以下4种通配符。
1.%：可匹配任意类型和长度的字符串。
2._（下划线）：可以匹配任何单个字符。
3.[]：指定范围或集合中的任何单个字符。
4.[^]：不属于指定范围或集合的任何单个字符。
例如：LIKE'刘%'匹配以'刘'开始的字符串；LIKE'%技术%'匹配的是前后字符为任意，中间含有“技术”两个字的字符串；LIKE'_秀%'匹配的是第二个字符为“秀”的任意字符串；[a-i] 匹配的是a，b，c，d，e，f，g，h，i单个字符；LIKE'm [^w-z]%'匹配的是以字母m开始并且第2个字母不为w，x，y，z的所有字符串。

例：
检索所有姓刘的学生的基本信息。分析：匹配所有姓刘的学生可以表示为：姓名LIKE'刘%'。

{% codeblock lang:SQL %}
USE JSJXY
GO
SELECT * FROM STUDENT
WHERE NAME LIKE '刘%'
GO
{% endcodeblock %}

正如所见，SQL 的通配符很有用。但这种功能是有代价的，即通配符搜索一般比前面讨论的其他搜索要耗费更长的处理时间。这里给出一些使用通配符时要记住的技巧。
1.不要过度使用通配符。如果其他操作符能达到相同的目的，应该使用其他操作符。
2.在确实需要使用通配符时，也尽量不要把它们用在搜索模式的开始处。把通配符置于开始处，搜索起来是最慢的。
3.仔细注意通配符的位置。如果放错地方，可能不会返回想要的数据。总之，通配符是一种极其重要和有用的搜索工具，以后我们经常会用到它


## 联合查询

联合查询是指将两个或两个以上SELECT语句，用UNION运算符连接起来的查询。联合查询可以将两个或更多查询的结果组合为单个结果集，该结果集包含联合查询中所有查询的全部行。使用UNION组合多个查询的结果时，必须注意：所有查询中的列数和列的顺序必须相同且数据类型必须兼容。


从成绩表中检索选修了“20001”或则“20002”的学生学号，合并两个查询的结果。

{% codeblock lang:SQL %}
SELECT ID FROM SC WHERE COURSEID='20001'
UNION 
SELECT ID FROM SC WHERE COURSEID='20002'
{% endcodeblock %}


## 连接查询
### 内连接
内连接（INNER JOIN）是组合两个表的常用方法，它将两个表中的列进行比较，将两个表中满足连接条件的行组合起来生成第3个表，仅包含那些满足连接条件的数据行。内连接有等值连接、自然连接和不等值连接3种。
当连接操作符是“=”时，该连接操作称为等值连接，使用其他运算符的连接运算称为不等值连接。当等值连接中的连接字段相同，并且在SELECT语句的<输出列表>中去除了重复字段时，该连接操作为自然连接。


查询姓名，课程名和成绩。

{% codeblock lang:SQL %}
SELECT STUDENT.ID, NAME, COURSENAME, SCORE 
FROM STUDENT 
INNER JOIN SC ON STUDENT.ID=SC.ID                 --连接条件
INNER JOIN COURSE ON COURSE.COURSEID=SC.COURSEID
{% endcodeblock %}


也可使用以下语句完成

{% codeblock lang:SQL %}
SELECT name,course.coursename,sc.score
FROM student,course,sc
WHERE student.id=sc.id AND course.courseid=sc.courseid
{% endcodeblock %}

### 外连接
在内连接中，只有在两个表中匹配的记录才能在结果集中出现。而外连接（OUTER JOIN）只限制一个表，而对另外一个表不加限制（即所有的行都出现在结果集中）。

外连接分为左外连接（LEFT [OUTER] JOIN）、右外连接（RIGHT [OUTER] JOIN）和全外连接（FULL [OUTER] JOIN）。括号中为使用FROM子句定义外连接的关键字，使用中可以省略OUTER。
#### 左外连接（LEFT OUTER JOIN）
左外连接对左边的表不加限制。左外连接需要在FROM子句中采用下列语法格式。
FROM 左表名 LEFT [OUTER] JOIN 右表名 ON 连接条件
查询是否所有的课程都有成绩，包括课程编号，课程名称，学号，成绩。
分析：所有的课程信息在课程表中，成绩信息在成绩表中，有关课程的成绩涉及两张表，由于要查询所有的课程信息，所以所有课程的信息都要出现在结果中，采用左外连接，左表为课程信息表。

{% codeblock lang:SQL %}
SELECT COURSE.COURSEID, COURSENAME, ID, SCORE 
FROM COURSE LEFT JOIN SC ON COURSE.COURSEID =SC.COURSEID
{% endcodeblock %}

#### 右外连接（RIGHT OUTER JOIN）
右外连接对右边的表不加限制。右外连接需要在FROM子句采用下列语法格式。
FROM 左表名 RIGHT [OUTER] JOIN 右表名 ON 连接条件

使用右外连接查询学生选修课程的信息。
分析：本查询涉及学生表和成绩，由于要显示所有学生选课信息，且用右外连接，所以学生表为右表。

{% codeblock lang:SQL %}
SELECT COURSEID,SCORE,NAME 
FROM SC RIGHT JOIN STUDENT
ON SC.ID=STUDENT.ID
{% endcodeblock %}

#### 全外连接（FULL OUTER JOIN）
全外连接对两个表都不加限制，即两个表中所有的行都会出现在结果集中。使用全外连接需要在FROM子句采用下列语法格式。
FROM 左表名 FULL [OUTER] JOIN 右表名 ON 连接条件
使用全外连接查询每个学生及其选修课程的情况（包括未选课的学生信息以及未被选修的课程信息）。

{% codeblock lang:SQL %}
SELECT STUDENT.ID,NAME,COURSE.COURSEID,COURSENAME,SCORE
FROM STUDENT FULL JOIN SC ON STUDENT.ID=SC.ID
FULL JOIN COURSE ON COURSE.COURSEID=SC.COURSEID
{% endcodeblock %}


#### 自连接
自连接就是一个表与它自身的不同行进行连接。因为表名要在FROM子句中出现两次，所以需要对表指定两个别名，使之在逻辑上成为两张表。
查找同名同姓的学生信息。
分析：该例是对学生表进行自连接，这里将学生表分别定义别名为A1、A2，将FROM子句写成“FROM STUDENT A1，STUDENT A2”，连接条件为“WHERE A1.NAME=A2.NAME AND A1.ID<>A2.ID”。为做本例题，需要修改STUDENT表中的某条数据，姓名改为一致。

{% codeblock lang:SQL %}
SELECT A1.* FROM STUDENT A1, STUDENT A2
WHERE A1.NAME=A2.NAME AND A1.ID<>A2.ID
{% endcodeblock %}

## 嵌套查询
例：
使用子查询求恰好有两门课程不及格的学生学号、姓名
{% codeblock lang:SQL %}
SELECT name
FROM student
WHERE id in
(
SELECT id
FROM sc
WHERE score<60
GROUP BY id having count(*)=2  
)
{% endcodeblock %}

## 分组查询
例：
查询每个学生的平均成绩，显示姓名和平均成绩。

{% codeblock lang:SQL %}
SELECT name,AVG(score) AS '平均成绩'
FROM student,sc
WHERE student.id = sc.id
GROUP BY sc.id,name  --先依据id，在根据姓名分组
{% endcodeblock %}


查询sc表中成绩超过科平均值的学生姓名。
{% codeblock lang:SQL %}
SELECT courseid ,AVG(score) AS '平均分'        --查询各科目成绩平均值
INTO avgsc
FROM sc
GROUP BY courseid

SELECT name
FROM student
WHERE id in                     --查找满足条件的id
(
SELECT id                     --查询单科成绩高于平均分的id
FROM sc,avgsc
WHERE score>平均分 and sc.courseid = avgsc.courseid 
)
DROP TABLE avgsc                  --删除临时创建的表
{% endcodeblock %}

## END


<a href="http://note.youdao.com/noteshare?id=428669b44b1ac7a5f01c29540c978c74&sub=52FA2A51E8274846A72B68142999903B"><blockquote class="blockquote-center">例题查看</blockquote></a>


{% note primary %}
参考资料
SQL入门经典第五版
SQL必知必会
{% endnote %}
