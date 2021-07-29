## Hive 报错处理

### 1. 关键字支持

##### 1.1 问题描述

在Hive2.1.0版本运行以下HQL时

~~~sql
create table order(
                      dt string,
                      order_id string,
                      user_id string,
                      amount decimal(10,2)
)row format delimited
fields terminated by '\t'
~~~

抛出如下异常:

~~~properties
Failed to recognize predicate 'order'. Failed rule: 'identifier' in table name
~~~

##### 1.2 问题分析

在Hive1.2.0版本开始增加了如下配置选项,默认值为true

~~~properties
hive.support.sql11.reserved.keywords
~~~

该选项的目的是: 是否启用对SQL2011保留关键字的支持,启用后,将支持部分SQL2011保留关键字

##### 1.3 解决方案

从上面知道报错原因是 order 是保留关键字, 所以直接使用导致报错,解决方案就是将该选项设置为false

~~~properties
hive.support.sql11.reserved.keywords = false
~~~

或者在conf下的hive-site.xml配置文件中修改配置选项

~~~properties
<property>
    <name>hive.support.sql11.reserved.keywords</name>
    <value>false</value>
</property>
~~~





