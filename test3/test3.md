# 实验3：创建分区表

## 一.实验目的：

掌握分区表的创建方法，掌握各种分区方式的使用场景。

## 二.实验内容：
- 本实验使用3个表空间：USERS,USERS02,USERS03。在表空间中创建两张表：订单表(orders)与订单详表(order_details)。
- 使用我自己的名字创建表，表创建在上述3个分区，自定义分区策略。
- 需要使用system用户给你自己的账号分配上述分区的使用权限。需要使用system用户给我的用户分配可以查询执行计划的权限。
- 表创建成功后，插入数据，数据能并平均分布到各个分区。每个表的数据都应该大于1万行，对表进行联合查询。
- 写出插入数据的语句和查询数据的语句，并分析语句的执行计划。
- 进行分区与不分区的对比实验。

## 三.实验步骤

实验三脚本文件:
首先创建自己的账号xuyue2，然后以system身份登录:

[student@deep02 ~]$sqlplus system/123@localhost/pdborcl
SQL>ALTER USER xuyue2 QUOTA UNLIMITED ON USERS;
SQL>ALTER USER xuyue2 QUOTA UNLIMITED ON USERS02;
SQL>ALTER USER xuyue2 QUOTA UNLIMITED ON USERS03;
SQL>exit

![1](..\test3\1.png)

在主表orders和从表order_details之间建立引用分区
在study用户中创建两个表：orders（订单表）和order_details（订单详表），两个表通过列order_id建立主外键关联。orders表按范围分区进行存储，order_details使用引用分区进行存储。
创建orders表的部分语句是：

```sql
SQL> CREATE TABLE orders 
(
 order_id NUMBER(10, 0) NOT NULL 
 , customer_name VARCHAR2(40 BYTE) NOT NULL 
 , customer_tel VARCHAR2(40 BYTE) NOT NULL 
 , order_date DATE NOT NULL 
 , employee_id NUMBER(6, 0) NOT NULL 
 , discount NUMBER(8, 2) DEFAULT 0 
 , trade_receivable NUMBER(8, 2) DEFAULT 0 
) 
TABLESPACE USERS 
PCTFREE 10 INITRANS 1 
STORAGE (   BUFFER_POOL DEFAULT ) 
NOCOMPRESS NOPARALLEL 
PARTITION BY RANGE (order_date) 
(
 PARTITION PARTITION_BEFORE_2016 VALUES LESS THAN (
 TO_DATE(' 2016-01-01 00:00:00', 'SYYYY-MM-DD HH24:MI:SS', 
 'NLS_CALENDAR=GREGORIAN')) 
 NOLOGGING 
 TABLESPACE USERS 
 PCTFREE 10 
 INITRANS 1 
 STORAGE 
( 
 INITIAL 8388608 
 NEXT 1048576 
 MINEXTENTS 1 
 MAXEXTENTS UNLIMITED 
 BUFFER_POOL DEFAULT 
) 
NOCOMPRESS NO INMEMORY  
, PARTITION PARTITION_BEFORE_2017 VALUES LESS THAN (
TO_DATE(' 2017-01-01 00:00:00', 'SYYYY-MM-DD HH24:MI:SS', 
'NLS_CALENDAR=GREGORIAN')) 
NOLOGGING 
TABLESPACE USERS02 
...
);
```

创建order_details表的部分语句如下：
```sql
SQL> CREATE TABLE order_details 
(
id NUMBER(10, 0) NOT NULL 
, order_id NUMBER(10, 0) NOT NULL
, product_id VARCHAR2(40 BYTE) NOT NULL 
, product_num NUMBER(8, 2) NOT NULL 
, product_price NUMBER(8, 2) NOT NULL 
, CONSTRAINT order_details_fk1 FOREIGN KEY  (order_id)
REFERENCES orders  (  order_id   )
ENABLE 
) 
TABLESPACE USERS 
PCTFREE 10 INITRANS 1 
STORAGE (   BUFFER_POOL DEFAULT ) 
NOCOMPRESS NOPARALLEL
PARTITION BY REFERENCE (order_details_fk1)
(
PARTITION PARTITION_BEFORE_2016 
NOLOGGING 
TABLESPACE USERS --必须指定表空间,否则会将分区存储在用户的默认表空间中
...
) 
NOCOMPRESS NO INMEMORY, 
PARTITION PARTITION_BEFORE_2017 
NOLOGGING 
TABLESPACE USERS02
...
) 
NOCOMPRESS NO INMEMORY  
);
```

![2](..\test3\2.png)

![3](..\test3\3.png)

## 四.查看数据库的使用情况

以下样例查看表空间的数据库文件，以及每个文件的磁盘占用情况。

$ sqlplus system/123@pdborcl
```sql
SQL>SELECT tablespace_name,FILE_NAME,BYTES/1024/1024 MB,MAXBYTES/1024/1024 MAX_MB,autoextensible FROM dba_data_files  WHERE  tablespace_name='USERS';
```
![4](..\test3\4.png)

```sql
SQL>SELECT a.tablespace_name "表空间名",Total/1024/1024 "大小MB",
 free/1024/1024 "剩余MB",( total - free )/1024/1024 "使用MB",
 Round(( total - free )/ total,4)* 100 "使用率%"
 from (SELECT tablespace_name,Sum(bytes)free
        FROM   dba_free_space group  BY tablespace_name)a,
       (SELECT tablespace_name,Sum(bytes)total FROM dba_data_files
        group  BY tablespace_name)b
 where  a.tablespace_name = b.tablespace_name;
```

![5](..\test3\5.png)

- autoextensible是显示表空间中的数据文件是否自动增加。
- MAX_MB是指数据文件的最大容量。

## 五.实验参考
- Oracle地址：202.115.82.8 用户名：system,hr,你的用户名 ， 密码123， 数据库名称：pdborcl，端口号：1521
- ssh oracle@202.115.82.8
- test3.sql

- SQL-DEVELOPER修改用户的操作界面：


- sqldeveloper授权对象的操作界面：
- ![授权](..\test3\授权.png)

## 六.实验总结:

​	

​		通过这次实验我掌握分区表的创建方法，掌握各种分区方式的使用场景。我们总共使用了三个表空间，用我自己的名字创建了这三张表，并且还学会了使用system用户来给我的账号分配权限，才能运行一些查询。表创建成功后，插入数据，数据能并平均分布到各个分区。每个表的数据都应该大于1万行，对表进行联合查询。分区表在我们今后工作和学习中都是有很大的用处的，它具有高可用性，一些表的分区故障，表的其他分区仍然可以使用并且维护还非常的方便，独立管理每个分区比管理单个达标要轻松的多，对于表的性能改善也非常的明显对大表的查询、增加、修改等操作可以分解到表的分区来并行执行，减小了单分区时的压力。在今后的学习中我们可能还会使用到分区表。





