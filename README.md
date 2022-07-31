# Wrr502.SQLNote
note
# SQL执行顺序和优化
1. FROM 和 JOINs
FROM 或 JOIN会第一个执行，确定一个整体的数据范围. 如果要JOIN不同表，可能会生成一个临时Table来用于 下面的过程。总之第一步可以简单理解为确定一个数据源表（含临时表)

2. WHERE
我们确定了数据来源 WHERE 语句就将在这个数据源中按要求进行数据筛选，并丢弃不符合要求的数据行，所有的筛选col属性 只能来自FROM圈定的表. AS别名还不能在这个阶段使用，因为可能别名是一个还没执行的表达式

3. GROUP BY
如果你用了 GROUP BY 分组，那GROUP BY 将对之前的数据进行分组，统计等，并将是结果集缩小为分组数.这意味着 其他的数据在分组后丢弃.

4. HAVING
如果你用了 GROUP BY 分组, HAVING 会在分组完成后对结果集再次筛选。AS别名也不能在这个阶段使用.

5. SELECT
确定结果之后，SELECT用来对结果col简单筛选或计算，决定输出什么数据.

6. DISTINCT
如果数据行有重复DISTINCT 将负责排重.

7. ORDER BY
在结果集确定的情况下，ORDER BY 对结果做排序。因为SELECT中的表达式已经执行完了。此时可以用AS别名.

8. LIMIT / OFFSET /  TOP
最后 LIMIT 和 OFFSET 从排序的结果中截取部分数据

## 简单优化
1、减少查询数据量，适用：查看列名、只看前几条数据的场景；

	查询表时，SQLServer、MySQL默认显示全部数据，oracle默认显示50行。
	SQLServer(top)：select top 100 * from...        (显示前100行) 
	           select top 50 PERCENT * from...      (显示前50%数据)
	MySQL(limit)：select * from...limit 100         (显示前100行)
	       select * from...limit 1,100              (忽略前1行，显示2-101行)
	Oracle(rownum)：select * from...rownum <= 100   (显示前100行)
2、SQLServer：
nolock和with(nolock)，加在表后使用。

	select * from table with(nolock)
	小区别:
	1)SQL05中的同义词，只支持with(nolock);
	2)with(nolock)的写法非常容易再指定索引;	
	3)跨服务器查询语句时,不能用with(nolock),只能用nolock;	
	同一个服务器查询时,则with(nolock)和nolock都可以用。
	
3、使用确定的
Schema: select * from dbo.tpatient(dbo.)

4、使用SET NOCOUNT ON(SET NOCOUNT OFF，可忽略)：
    
    SET NOCOUNT ON  
	select * from dbo.tpatient(dbo.)	
	SET NOCOUNT OFF	
	SET NOCOUNT ON：只影响当前窗口或者当前存储过程，关闭当前窗口后会自动变OFF

5、创建索引：create index index_name on table_name column_name
	
	CREATE INDEX PersonIndex ON Person (LastName DESC);
	CREATE INDEX PersonIndex ON Person (LastName, FirstName)
	
