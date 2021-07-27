### 店铺访问UV,top3

- 有50W个京东店铺，每个顾客访客访问任何一个店铺的任何一个商品时都会产生一条访问日志，

  访问日志存储的表名为Visit，访客的用户id为user_id，被访问的店铺名称为shop，数据如下：

~~~sql
user_id	shop
u1	a
u2	b
u1	b
u1	a
u3	c
u4	b
u1	a
u2	c
u5	b
u4	b
u6	c
u2	c
u1	b
u2	a
u2	a
u3	a
u5	a
u5	a
u5	a
~~~

- 建表

~~~sql
create table visit_shop(
	id string,
    shop string
)row format delimited
fields terminated by '	'
~~~

- 加载数据

~~~sql
load data local inpath '/export/data/hive_exercise/visit_shop.txt' into table visit_shop
~~~

- 需求1: 每个店铺的UV(访客数)

~~~properties
# 需求解析
这里需要注意的一点是, 一个用户的多次访问同一家店铺,只算一次访问
~~~

- 答案

~~~sql
select shop,
       count(distinct id)
from visit_shop
group by shop
~~~

- 需求2: 每个店铺访问次数top3的访客信息,输出店铺名称,访客id,访问次数

~~~properties
#需求解析
1. 首先, 求解出每个店铺,每个用户的访问次数 记为t
2. 使用排序窗口函数, 将用户uv排好序 记为t2
3. 取出t2 中排序前三的访客信息
~~~

- 答案

~~~sql
select t2.shop,t2.id,t2.uv
from (
         select t.shop,
                t.id,
                t.uv,
                row_number() over (partition by t.shop order by t.uv desc) rn
         from (
                  select shop,
                         id,
                         count(1) uv
                  from visit_shop
                  group by shop,id
              ) t
         ) t2
where t2.rn <= 3

~~~



