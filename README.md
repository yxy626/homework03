# 习题一

## 定义语句

create table acquaintance(

​    -> friend1 VARCHAR(200) not null,

​    -> friend2 VARCHAR(200) NOT NULL,

​    -> CLASS VARCHAR(200) NOT NULL);

![](C:\Users\oysd\Desktop\数据库2.png)

![](C:\Users\oysd\Desktop\数据库1.png)

## 生成测试数据

我用的是excel生成测试数据。

friend1和friend2运用的函数是“=CHAR(INT(RAND())32+RANDBETWEEN(65,75))&CHAR(65)*”，*随机生成了“首字母是A-K，第二个字母是A”的数据。

Class使用的函数是“=CHAR(INT(RAND())32+RANDBETWEEN(75,80))&CHAR(95)&CHAR(70)”，随机生成了“K-P任意字母+下划线+F”的数据。

下拉，生成100条数据。

最后，生成CSV文件，导入phpmyadmin页面。

![](C:\Users\oysd\Desktop\数据库3.png)

# 习题二

## 找出互不认识的人

```
with t as (
select friend1 m from acquaintance
union 
select friend2 from acquaintance
)
select t1.m m1, t2.m m2
from t t1, t t2
where t1.m!=t2.m
and not exists (select * from acquaintance where (friend1=t1.m and friend2=t2.m) or (friend2=t1.m and friend1=t2.m))
```

## 找出只在一个类别里出现的人

```
with t as(
SELECT friend1 AS m, class FROM acquaintance
UNION
SELECT friend2, class FROM acquaintance
)
SELECT t1.m
FROM t AS t1
WHERE NOT EXISTS(SELECT * FROM t WHERE m=t1.m AND class<>t1.class)
```

## 找出在所有类别里都有朋友的人

```
with t as(
SELECT friend1 AS m, class FROM acquaintance
UNION
SELECT friend1 ,class FROM acquaintance
)
SELECT m
FROM t AS t1
WHERE m=t1.m AND NOT EXISTS(SELECT * FROM t WHERE t.class<>t1.class);
```

## 找出每个类别里面朋友最多的人

```
with t as (
select M,class,sum(cn) as cnt from 
(select friend1 AS M,class,count(*) as cn from acquaintance group by friend1,class
union all
select friend2 AS M,class,count(*) as cn from acquaintance group by M,class)
as t group by M,class)
select * from t AS t1 where cnt=(select max(cnt) from t where t1.class=t.class)
ORDER BY class
```

## 找出在同一类别里面通过朋友而结识的其他朋友（朋友的朋友也是朋友）

没做出来qwq

## 找出这样的人，通过他而结识的朋友对最多(p1和p2原本不相识，他们通过p3结识，那么p3的连接度为1，找出连接度最高的人)

```
 with t as 
(
select friend1,friend2
from   acquaintance
union all
select friend2,friend1
from   acquaintance
),
related as
(
select t1.friend1,
       t1.friend2 as midman,
       t2.friend2 
from   t as t1
       inner join t as t2
         on t1.friend2 = t2.friend1
)
select midman,count(*) as rlt_index
from (
        select *
        from   related as rlt
        where  rlt.friend1 <> rlt.friend2
     ) a group by midman		 
ORDER BY rlt_index DESC
```

## 找出臭味相投的朋友，他们在所出现的所有类别里面都是朋友（并且不能有这种情况，其中一个在某个类别里出现而另外一个不出现）

```
select friend1,friend2,class from acquaintance as t1
where not exists (select * from acquaintance as t2
where not exists(
select * from acquaintance 
where class=t2.class and ((friend1=t1.friend1 and friend2=t1.friend2) or (friend1=t1.friend2 and friend2=t1.friend1))))
```

