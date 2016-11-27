---
layout: post
title: MySQL查询优化
description: "MySQL查询优化方法，MySQL如何优化查询，MySQL怎么优化查询，如何对MySQL进行查询优化，MySQL优化查询方式"
modified: 2016-10-14
tags: [mysql]
categories: [intro]
---
##优化数据访问

###是否向数据库请求了不需要的数据

例如：

-查询不需要的记录。（使用limit限制）

-多表关联时返回全部列。

-总是取出全部列。

-重复查询相同的数据。

###是否在扫描额外的记录

-响应时间包括服务时间和排队时间。

-查看扫描的行数和返回的行数来判断查询效率。

-根据扫描的行数和访问类型来判断查询傲率。如果查询没有办法找到合适的访问类型，通常的方法时增加一个合适的索引。

MySQL能够使用如下三种方式应用WHERE条件，从号到坏依次为

1、在索引中使用WHERE条件来过滤不匹配的记录（在存储引擎完成）；

2、使用索引覆盖扫描来返回记录，直接从索引中过滤不需要的记录并返回命中的结果。

3、从数据表中返回数据，然后过滤不满足条件的记录。

如果发现查询需要扫描大量的数据但是只返回少数的行，那么通常可以尝试采取以下措施：

1、使用索引覆盖扫描，把所有需要用的列都放到索引中。

2、改变表的结构。

3、重写查询语句，让MySQL优化器能够以更优化的方式执行这个查询。

<!-- more -->

##重构查询的方式

-是否需要将一个复杂的查询分成多个简答的查询。

-有时候需要将大查询切分成小查询，每个查询功能完全一样，只完成一小部分，每次只返回一小部分查询结果。

-用分解关联查询的方式重构查询的优势：

1、让缓存的效率更高。

2、将查询分解后，执行单个查询可以减少锁的竞争。

3、在应用层做关联，可更容易对数据库进行拆分，更容故意做到高性能和高扩展性。

4、查询本身效率也可能会有所提升。

5、可以减少冗余记录的查询。意味着多余某条记录应用只需要查询一次，而在数据库中做关联查询，则可莪能需要重复的访问一部分数据。

6、这样做相当于在应用中实现了哈希关联，而不是使用MySQL的嵌套循环关联。

##查询执行的过程

如图

<figure class="center">
	<a href="#"><img src="{{ site.baseurl }}/images/optimize-query-1.jpg" alt=""></a>
</figure>

1、客户端先发送一条查询给服务器。

2、服务器先检查查询缓存，如果命中缓存，则立即返回存储在缓存中的结果。否则进行下一阶段。

3、服务器端进行SQL解析、预处理，再由优化器生成对应的执行计划。

4、MySQL根据优化器生成的执行计划，调用存储引擎的API来执行查询。

5、返回结果给客户端。

（MySQL客户端与服务器之间的通信协议是“半双工”。）

##MySQl查询优化器的局限性

###关联子查询

关联子查询的实现非常糟糕。最糟糕的一类查询是WHERE条件中包含IN()的子查询语句。

例如

{% highlight ruby %}

SELECT * FROM film WHERE film_id IN(
	SELECT film_id FROM film_actor WHERE actor_id = 1);

{% endhighlight %}

MySQL不会有限执行子查询，而是将相关的外层表压倒子查询中。会变为如下形式：

{% highlight ruby %}

SELECT * FROM film WHERE EXISTS(
	SELECT * FROM film_actor WHERE actor_id = 1
	AND film_actor.film_id = film.film_id);

{% endhighlight %}

###UNION的限制

UNION查询会把多个表中的所有数据饭在一个临时表中，然后再从临时表中取出所需条数。如果希望UNION的各个子句能够根据LIMIT只取部分结果集，或者希望能够先排好序在合并结果集的话，就需要在UNION的各个子句中分别使用这些子句。

###索引合并优化

在WHERE子句中包含多个复杂条件的时候，MySQL能够访问单个表的多个索引以合并和交叉过滤的方式来定位需要查找的行。

###等值传递

某些时候，等值传递会带来一些意想不到的额外消耗。例如，有一个非常大的IN（）列表，并存在WHERE、ON或USING子句，将这个列表的值与另外某个表的列相关联。那么优化器会将IN（）列表都复制应用到关联的各个表中。通常，各个表新增了过滤条件，优化器可以更高效地从存储引擎过滤记录。但如果词列表非常大，则会导致优化和执行都会变慢。

###并行执行

MySQL无法利用多核特性来并行执行查询。

###哈希关联

MySQL的所有关联都是嵌套循环关联，不支持哈希关联，但可以通过建立一个哈希索引来曲线地实现哈希关联。

###松散索引扫描

MySQL不支持松散索引扫描，也就无法按照不连续的方式扫描一个索引。MySQL的索引扫描需要新定义一个起点和重点，即使需要的数据只是这段索引中很少数的几个，MySQL仍需要扫描这段索引中的每一个条目。

###最大值和最小值优化

如果在某个字段上并没有索引，因此MySQL将会进行一次全表扫描。可用LIMIT来重写MIN()查询。

{% highlight ruby %}

SELECT id from test USE INDEX(PRIMARY)
WHERE  name = 'name' LIMIT 1;

{% endhighlight %}

###在同一个表上查询和更新

MySQL不允许对同一张表同时进行查询和更新。可通过生成表的形式来绕过上面的限制。

{% highlight ruby %}

UPDATE table SET cnt =( SELECT ....)

修改为

UPDATE table INNER JOIN(
	SELECT ..)AS der USING(type)
SET table.cnt = der.cnt;

{% endhighlight %}

##优化特定类型的查询

###优化COUNT()查询

COUNT()函数既可以统计某个列值的数量，也可以统计行数。在统计列值是要求列值是非空的（不统计NULL）。在统计行数的时候通常使用COUNT(*)，较为常见的一个错误是，在括号内指定一个列却希望统计结集的行数。

当估计COUNT()总数很大的时候，可以使用COUNT()的补集来计算。

若同时计算不同列的总数可使用
{% highlight ruby %}

SELECT SUM(IF(column = 'first',1,0))AS first,SUM(IF(column = 'second',1,0)) AS second FROM table;

或

SELECT COUNT(column = 'first' OR NULL) AS first,COUNT(column = 'second' OR NULL) AS second FROM table;

{% endhighlight %}

很多时候，计算精确值的成本非常高，而计算近似值则非常简单。EXPLAIN执行时并不需要真正去执行查询，因此EXPLAIN出来的优化器估算的行数得到的近似值效率很高。而进一步优化则可以尝试删除DISTINCT这样的约束来避免文件排序。

“快速，精确和实现简单”，三者永远只能满足其二，必须舍掉其中之一。

###优化关联查询

-确保ON或者USING子句中的列上有索引。没有用的索引只会带来额外的负担。

-确保任何的GROUP BY和ORDER BY中的表达式只涉及到一个表中的列，这样MySQL才有可能使用索引来优化这个过程。

-升级MySQL的时候需要注意：关联语法、运算符优先级等其他可能会发生变化的地发。

### 优化子查询

尽可能使用关联查询代替。而MySQL5.6以上的版本或MariaDB，则可以直接忽略此建议。

###优化GROUP BY和DISTINCT

如需对关联查询做GROUP BY操作，那么采用表的标识列分组的效率会比其他列更高。

在分组查询的SELECT中直接使用非分组列的结果通常是不定的，当索引改变，或者优化器选择不同的优化策略时可能导致结果不一样。

分组查询的一个变种就是要求MySQL对返回的分组结果再做一次超级聚合。可使用WITH ROLLUP来实现，但可能会不够优化。最好的办法是尽可能的将WITH ROLLUP功能转移到应用程序中处理。

###优化LIMIT分页

若偏移量非常大的时候，例如LIMIT1000，20这样的查询，MySQL需要查询0020条记录然后只返回最后20条，前面1000条将被抛弃。因此可进行如下优化：
{% highlight ruby %}

SELECT id,description FROM film ORDER BY title limit 50,50;

改为

SELECT id, description FROM fillm 
	INNER JOIN(
		SELECT id FROM film ORDER BY title LIMIT 50,5
	)AS lim USING(id);

{% endhighlight %}

通过以上的“延迟关联”，MySQL在获取需要访问的记录后再根据关联列返回原表查询需要的所有列。

灵位还可用将LIMIT转换为已知位置的查询，使用BETWEEN AND 

###优化SQL_CALC_FOUND_ROWS

分页的时候，在LIMIT语句中加上SQL-CALC_FOUND_ROWS提示(hint)，从而去掉LIMIT以后满足条件的行数。但加上这个提示后，不管是否需要，MySQL都会扫描所有满足条件的行，然后再抛弃不需要的行，而不是满足LIMIt的行数后就终止扫描。因此该提示的代价可能很高。

一种分页方法是假设每页20条记录，则LIMIT 21查询21条，若存在最后一条，则显示下一页。另一种方法是利用缓存。

###优化UNION查询

除非确实需要服务器消除重复的行，否则就一定要使用UNION ALL，如果没有ALL关键字，MySQL会给临时表加上DISTINCT选项，则会导致对整个临时表的数据作为一性检查，而这样做的代价非常高。其他方法则包括手工地将WHERE、LIMIT\ORDER BY等子句“下推”到UNION的各个子查询中，以便优化器可以充分利用这些条件进行优化。

###静态查询分析

Percona Toolkit中的**pt-query-advisor**能够解析查询日志、分析查询模式，然后给出所有可能存在潜在问题的查询，并给出建议。

###使用用户自定义变量

例如：
{% highlight ruby %}

SET @one       := 1;

SET @mian-actor:= (SELECT MIN(actor_id) FROM actor);

SET @last_week := CURRENT_DATE-INTERVAL 1 WEEK;

SELECT ...WHERE col <= @last_week;

{% endhighlight %}

不能使用的场景：

-使用自定义变量的查询，无法使用查询缓存。

-不能再使用常量或标识符的地方使用自定义变量，如表名、列明和LIMIT子句中。

-用户自定义变量的生命周期只在一个连接中有效。

-5.0版本之前大小写敏感。

不能显示的声明自定义变量的类型。

-MySQL优化器在某些场景下可能会将这些变量优化掉。

-赋值的顺序和赋值的时间点并不总是固定的。

-赋值符号：=的优先级非常低。

-使用未定义变量不会产生人和语法错误。


使用原则

####优化排名语句

使用用户自定义变量的重要特征时可以在给一个变量赋值的同时使用这个变量。

####避免重复查询刚刚跟新的数据。

{% highlight ruby %}

UPDATE ti SET lastupdated = NOW() WHERE id = 1 AND @ now := NOW();

SELECT @now;

{% endhighlight %}

####统计更新和插入的数量

当时用INSERT ON DUPLICATE KEY UPDATE的时候，如果想知道都低插入了多少行数据，到底有多少数据是因为冲突而改写成更新操作的？一个解决办法为：
{% highlight ruby %}

INSERT INTO t1(c1,c2) VALUES(4,4),(2,1),(3,1) ON DUPLICATE KEY UPDATE

c1 = VALUES(c1) + (0 * (@x := @x + 1));

{% endhighlight %}

####确定取值的顺序

使用用户自定义变量的一个最常见的文艺就是没有注意到在赋值的读取变量的时候可能是在查询的不同阶段。可使用内置函数LEAST()、ISNULL()、NULLIFL()、IF()等对变量进行判断。

####编写偷懒的UNION

例如

{% highlight ruby %}

SELECT id FROM users WHERE id = 123

UNION ALL

SELECT id FROM users_archived WHERE id = 123;

{% endhighlight %}

上面的查询可以正常工作，但即使前面找到了id为123的用户，后面还会在进行遍历，使用自定义变量可以采用“偷懒”的方式，使得避免产生额外的数据。

{% highlight ruby %}

SELECT GREATEST(@found := -1,id) AS id,'users' AS which_tbl FROM users WHERE id = 1

UNION ALL

SELECT id,'users_archived' FROM users_archived WHERE id = 1 AND @found IS NULL 

UNION ALL

SELECT 1,'reset' FROM DUAL WHERE(@found := NULL) IS NOT NULL;

{% endhighlight %}

####用户自定义变量的其他用处

-查询运行时计算总数和平均值

模拟GROUP语句中的函数FIRST()和LAST()

-对大量数据做一些数据计算

-计算一个大表的MD5散列值

-编写一个样本处理函数，当样本中的数值超过某个边界值的时候将其变为0.

-模拟读/写游标。

-在SHOW语句的WHERE子句中加入变量值。