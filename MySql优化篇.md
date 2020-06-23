# MySql基础和优化

[toc]

---

## 一、基础篇

### 1.单表查询

----

#### 1.将空值转换为实际值

+ `IFNULL`：相当于oracle的nvl

```mysql
select IFNULL(t.item_type,123),t.* from item t ;
```

+ `COALESCE`：`coalecse(表达式1,表达式2,表达式3...)`
  + **`表达式1`为null，判断`表达式2`，`表达式2`为null，判断`表达式3`，不为null，则返回表达式3的值**

```mysql
select COALESCE(t.item_type,t.batch_id,item_description,sh_id),t.* from item t ;
```

#### 2.字符串拼接

+ `concat`：`concat(字符串1,字符串2,字符串3)`

```mysql
select concat(COALESCE(t.item_type,t.batch_id,item_description,sh_id),"—",item_name),t.* from item t ;
```

#### 3.在Select语句中使用条件逻辑

+ `CASE WHEN`：在表达式里执行条件判断

```mysql
select item_name,
	(CASE
     	WHEN '表达式1' THEN '结果1'
		WHEN '表达式2' THEN '结果2'
     	WHEN '表达式3' THEN '结果3'
     	ELSE '缺省值'
    END) as '名称'
from item;
```

#### 4.限制返回的行数

+ `limit`：限制查询行数，
  + `limit 0,4`：从第**1**行开始，显示**4**条
  + `limit 4`：与上面效果一样
  + `limit 4,1`：从第**4**行开始，显示**1**条

```mysql
select * from film LIMIT 5;
select * from film LIMIT 5,10;
```

#### 5.模糊查询

+ `like`：`like '%ABC'`，**后面总结sql优化**
  + `%`：表示替代（一个或多个字符）
  + `_`：标识替代（一个或多个字符）
+ `ESCAPE`：当'`_`'被当通配符了，而我们需要查的数据是：`_ABC%`时，需要用转义字符，`ESCAPE '/'`标识转义符

```mysql
# 
select * from actor t where t.first_name like '%/_U%' ESCAPE '/';
```

---

### 2.查询结果排序

----

+ `order by`：`order by asc` 或 `order by desc`
  + `order by 2 asc`：该形式表示第2列升序，也等价于 `ORDER BY 4`
  + `order by 2 asc,3 DESC`：该形式表示第2列升序，第3列降序

---

### 3.多个表操作

---

#### 1.UNION ALL和UNION

+ `union all`：用于两张表的合并数据，**字段必须相同才可以合并**

```mysql
#方式1:
select '张三' as name,'14' as age,'女' as sex from dual
union all
select '李四' as name,'24' as age,'男' as sex from dual;
#方式2
select '张三' as name,'14' as age,'女' as sex from dual
union all
select '李四' as name,'' as age,null as sex from dual;
```

+ `union`：合并两张表，但**会过滤掉，重复数据**

```mysql
#形式1
select '张三' as name,'14' as age,'男' as sex from dual
union all
select '张三' as name,'14' as age,'男' as sex from dual;
#结果1
张三	14	男
张三	14	男
####################################################################
#形式2
select '张三' as name,'14' as age,'男' as sex from dual
union 
select '张三' as name,'14' as age,'男' as sex from dual;
#结果2
张三	14	男
```

+ 查询条件是否走索引

```mysql
#建立索引
create index idx_film_title on film (title);
create index idx_film_last_update on film (last_update);
#查询是否走索引
explain select * from film t where t.last_update ='2006-02-15 05:03:41' or t.title = 'ACE GOLDFINGER';
#修改后
explain 
	select * from film t where t.last_update ='2006-02-15 05:03:42'
union 
 	select * from film t where t.title = 'ACE GOLDFINGER';


```

#### 2.IN、EXISTS 和 INNER JOIN

#### 3.NOT IN、NOT EXISTS 和 LEFT JOIN

#### 4.INNER JOIN 、LEFT JOIN 、RIGHT JOIN 和 FULL JOIN

