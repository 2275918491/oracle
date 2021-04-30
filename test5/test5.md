# 实验5：PL/SQL编程

## 实验目的：

```
了解PL/SQL语言结构
了解PL/SQL变量和常量的声明和使用方法
学习条件语句的使用方法
学习分支语句的使用方法
学习循环语句的使用方法
学习常用的PL/SQL函数
学习包，过程，函数的用法。
```

## 实验场景：

- 假设有一个生产某个产品的单位，单位接受网上订单进行产品的销售。通过实验模拟这个单位的部分信息：员工表，部门表，订单表，订单详单表。
- 本实验以实验四为基础

## 实验内容：

- 创建一个包(Package)，包名是MyPack。

```sql
create or replace PACKAGE MyPack IS
  FUNCTION SaleAmount(departmen_id number) RETURN NUMBER;
END MyPack;
```

- 在MyPack中创建一个函数SaleAmount ，查询部门表，统计每个部门的销售总金额，每个部门的销售额是由该部门的员工(ORDERS.EMPLOYEE_ID)完成的销售额之和。函数SaleAmount要求输入的参数是部门号，输出部门的销售金额。

```sql
create or replace PACKAGE BODY MyPack IS
  FUNCTION SaleAmount(departmen_id NUMBER) RETURN NUMBER
  AS
    sales_amount NUMBER(20,2); 
    BEGIN
      select sum(product_num*product_price) into sales_amount from order_details where order_id in(
        select order_id from orders where employee_id in(
            select employee_id from employees where department_id in (DEPARTMENT_ID)));
      RETURN sales_amount;
    END;
END MyPack;
```



- 在MyPack中创建一个过程，在过程中使用游标，递归查询某个员工及其所有下属，子下属员工。过程的输入参数是员工号，输出员工的ID,姓名，销售总金额。信息用dbms_output包中的put或者put_line函数。输出的员工信息用左添加空格的多少表示员工的层次（LEVEL）。比如下面显示5个员工的信息：

- ```sql
  create or replace PACKAGE MyPack IS
    FUNCTION Get_SaleAmount(V_DEPARTMENT_ID NUMBER) RETURN NUMBER;
    PROCEDURE Get_Employees(V_EMPLOYEE_ID NUMBER);
  END MyPack;
  create or replace PACKAGE BODY MyPack IS
    FUNCTION SaleAmount(departmen_id NUMBER) RETURN NUMBER
    AS
      sales_amount NUMBER(20,2); 
      BEGIN
        select sum(product_num*product_price) into sales_amount from order_details where order_id in(
          select order_id from orders where employee_id in(
              select employee_id from employees where department_id in (DEPARTMENT_ID)));
        RETURN sales_amount;
      END;
  
    PROCEDURE GET_EMPLOYEES(V_EMPLOYEE_ID NUMBER)
    AS
      LEFTSPACE VARCHAR(2000);
      begin
        --通过LEVEL判断递归的级别
        LEFTSPACE:=' ';
        --使用游标
        for v in
        (SELECT LEVEL,EMPLOYEE_ID,NAME,MANAGER_ID FROM employees
        START WITH EMPLOYEE_ID = V_EMPLOYEE_ID
        CONNECT BY PRIOR EMPLOYEE_ID = MANAGER_ID)
        LOOP
          DBMS_OUTPUT.PUT_LINE(LPAD(LEFTSPACE,(V.LEVEL-1)*4,' ')||
                               V.EMPLOYEE_ID||' '||v.NAME);
        END LOOP;
      END;
  END MyPack;
  /
  ```


## 实验结果：

函数Get_SaleAmount()测试方法：

```sql
select count(*) from orders;
select MyPack.Get_SaleAmount(11) AS 部门11应收金额,MyPack.Get_SaleAmount(12) AS 部门12应收金额 from dual;
```

![函数测试方法1](image\函数测试方法1.png)

过程Get_Employees()测试代码：

```sql
set serveroutput on
DECLARE
  V_EMPLOYEE_ID NUMBER;    
BEGIN
  V_EMPLOYEE_ID := 1;
  MYPACK.Get_Employees (  V_EMPLOYEE_ID => V_EMPLOYEE_ID) ;  
  V_EMPLOYEE_ID := 11;
  MYPACK.Get_Employees (  V_EMPLOYEE_ID => V_EMPLOYEE_ID) ;    
END;
/
```

![函数测试方法2](image\函数测试方法2.png)

## 实验总结：

通过这次实验我了解PL/SQL语言结构和PL/SQL变量和常量的声明和使用方法学会了条件语句、分支语句、循环语句、PL/SQL函数以及包，过程，函数的用法。存储过程和存储函数是事先经过编译并存储在数据库中的一段SQL语句的集合，调用存储过程和函数可以简化应用开发人员的工作，减少数据在数据库和应用服务器之间的传输，能够提高数据库的处理效率。存储过程在高并发数据中使用的比较多。存储过程和存储函数的区别在于存储函数是必须有返回值的，而存储过程并没有。存储过程的参数可以是in ，out ，inout类型，存储函数的参数类型只能是in。所以可以说存储函数是一个有返回值的存储过程，而存储过程是一个没有返回值的存储函数。我们还是血药多多练习存储过程以及函数的用法。