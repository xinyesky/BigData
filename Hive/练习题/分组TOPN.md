### 分组TOPN

场景举例: 北京市学生成绩分析

成绩的数据格式: 时间, 学校, 年级, 姓名, 科目,成绩（该数据样例不足）

~~~sql
2013,北大,1,裘容絮,语文,97
2013,北大,1,庆眠拔,语文,52
2013,北大,1,乌洒筹,语文,85
2012,清华,0,钦尧,英语,61
2015,北理工,3,冼殿,物理,81
2016,北科,4,况飘索,化学,92
2014,北航,2,孔须,数学,70
2012,清华,0,王脊,英语,59
2014,北航,2,方部盾,数学,49
2014,北航,2,东门雹,数学,77
~~~
- 建表

~~~sql
create table student(
	year string,
    school string,
    grade int,
    name string,
    class string,
    score int
)row format delimited
fields terminated by ','
~~~

- 数据加载

~~~sql
load data local inpath '/export/data/hive_exercise/student.txt' into table student
~~~

- 需求1: 分组TOPN选出, 今年每个学校, 每个年级, 分数前三的科目

~~~properties
# 需求解析:
1. 首先根据学校, 年级进行分组
2. 使用窗口排序 对分数进行排名
3. 取排名<=3的记录
备注: 对于分数前三的科目这个需求, 说实话我觉得不够明确,从答案反推, 我觉得需求应该表示为: 分组TOPN选出, 今年每个学校, 每个年级, 每个科目的分数前三名
~~~

- 涉及知识
  - row_number 函数: row_number() 按指定的列进行分组生成行序列,从1开始, 如果两行记录的分组列相同, 则行序列+1
  - over 函数: 是一个窗口函数
    - over(order by score) 按照score排序进行累计, order by是个默认的开窗函数
    - over(parittion by class) 按照班级分区
    - over(parition by class order by score) 按照班级分区,并按着分数排序
    - over( order by score range between 2 preceding and 2 following) : 窗口范围为当前行的数据幅度减2及加2的范围内的数据求和

- 答案

~~~sql
select t.school,
       t.grade,
       t.class,
       t.grade,
       t.rn
from(
        select school,
               grade,
               class,
               score,
               row_number() over (partition by school,grade,class order by score) rn
        from student
    	#where year=‘某年’
        ) t
where t.rn <= 3
~~~

- 需求2 ： 今年，北航， 每个班级，每科的分数，及分数上下浮动2分的总和

~~~properties
# 需求解析： 
求浮动的综合， 需要用到 over( order by score range between 2 preceding and 2 following)

解决思路：
筛选条件： 北航
分组：班级，科目
窗口范围： -2~2
~~~

- 答案

~~~sql
# 个人答案
select school,
       grade,
       class,
       score,
       sum(score) over (partition by grade,class order by score range between 2 preceding and 2 following)
from student
where school='北航'

#原始答案
select school,class,subjects,score,
sum(score) over(order by score range between 2 preceding and 2 following) sscore
from spark_test_wx
where partition_id = "2017" and school="北航"
~~~

- 之所以我没有对年份进行判断， 是因为数据样例太少， 所以就不进行年份筛选