# 实验4：对象管理

### 实验目的：

了解Oracle表和视图的概念，学习使用SQL语句Create Table创建表，学习Select语句插入，修改，删除以及查询数据，学习使用SQL语句创建视图，学习部分存储过程和触发器的使用。

### 实验场景：

假设有一个生产某个产品的单位，单位接受网上订单进行产品的销售。通过实验模拟这个单位的部分信息：员工表，部门表，订单表，订单详单表。

1.创建部门表![2.创建部门表](C:\Users\22759\Desktop\实验四\image\2.创建部门表.png)

2.创建员工表![3.创建员工表](C:\Users\22759\Desktop\实验四\image\3.创建员工表.png)

3.创建订单表![4.创建order表](C:\Users\22759\Desktop\实验四\image\4.创建order表.png)

4.创建临时中间表![5.创建临时表](C:\Users\22759\Desktop\实验四\image\5.创建临时表.png)

## 实验内容：

### 录入数据：

要求至少有1万个订单，每个订单至少有4个详单。至少有两个部门，每个部门至少有1个员工，其中只有一个人没有领导，一个领导至少有一个下属，并且它的下属是另一个人的领导（比如A领导B，B领导C）。

![9.插入数据](C:\Users\22759\Desktop\实验四\image\9.插入数据.png)

###  序列的应用

插入ORDERS和ORDER_DETAILS 两个表的数据时，主键ORDERS.ORDER_ID, ORDER_DETAILS.ID的值必须通过序列SEQ_ORDER_ID和SEQ_ORDER_ID取得，不能手工输入一个数字。

![6.创建序列](C:\Users\22759\Desktop\实验四\image\6.创建序列.png)

###  触发器的应用：

维护ORDER_DETAILS的数据时（insert,delete,update）要同步更新ORDERS表订单应收货款ORDERS.Trade_Receivable的值。

![8.创建行级触发器](C:\Users\22759\Desktop\实验四\image\8.创建行级触发器.png)

创建行级触发器![8.创建行级触发器](C:\Users\22759\Desktop\实验四\image\8.创建行级触发器.png)



## 创建索引：

索引是与表相关的一个可选结构，用以提高 SQL 语句执行的性能，减少磁盘I/O

![10.创建索引](C:\Users\22759\Desktop\实验四\image\10.创建索引.png)

###  查询数据：

##### 1.查询索引

```sql
select * from USER_IND_PARTITIONS;
```

![1.1查询索引](C:\Users\22759\Desktop\实验四\image\1.1查询索引.png)

##### 2.查询一个分区

```sql
select count(*) from ORDER_DETAILS partition(PARTITION_BEFORE_2016);
select count(*) from ORDER_DETAILS partition(PARTITION_BEFORE_2017);
```

![1.2](C:\Users\22759\Desktop\实验四\image\1.2.png)

##### 3.查询表空间使用情况

```sql
SELECT a.tablespace_name "表空间名",
total "表空间大小",
free "表空间剩余大小",
(total - free) "表空间使用大小",
total / (1024 * 1024 * 1024) "表空间大小(G)",
free / (1024 * 1024 * 1024) "表空间剩余大小(G)",
(total - free) / (1024 * 1024 * 1024) "表空间使用大小(G)",
round((total - free) / total, 4) * 100 "使用率 %"
FROM (SELECT tablespace_name, SUM(bytes) free
FROM dba_free_space
GROUP BY tablespace_name) a,
(SELECT tablespace_name, SUM(bytes) total
FROM dba_data_files
GROUP BY tablespace_name) b
WHERE a.tablespace_name = b.tablespace_name
```

![1.3](C:\Users\22759\Desktop\实验四\image\1.3.png)

## 实验报告：

本次实验的主要内容是：创建表，触发器，创建和使用序列，序列在插入大量数据时可以对主键或者有序列号要求的赋值。并且还学会了使用SQL语句Create Table创建表，Select语句插入，修改，删除以及查询数据，使用SQL语句创建视图，部分存储过程和触发器的使用。序列可以自动为数据生成列号。创建表和触发器，可能是我们工作之后非常重要且基本的操作，对于这一部分的内容我们必须要做到烂熟于心。







