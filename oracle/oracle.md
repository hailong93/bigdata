[TOC]

# Sql

## 系统表相关

### 用户操作

* 查看用户下所有的表

  ~~~sql
  select * from user_tables;
  ~~~

* 用户的锁定和解锁

  ~~~~plsql
  alter user [username] account lock;
  alter user [username] account unlock;
  ~~~~

* 修改密码

  ~~~plsql
  alter user [username] identified by password;
  ~~~

* 查看用户表信息

  ~~~plsql
  select * from dba_users;
  ~~~

* 创建用户


  ~~~plsql
  create user [user] identified by password;
  ~~~

* 删除用户

  ~~~plsql
  drop user [username] cascade;
  ~~~

  

### 简单赋权

1. 权限的赋予与收回

   ~~~plsql
   grant [权限] to [用户];
   
   revoke [权限] from [用户];
   ~~~

2. 查看当前用户的所用权限

   ~~~plsql
   select * from session_privs;
   ~~~

3. 查看用户的缺省表空间

   ~~~plsql
   select username,default_tablespace from user_users;
   
   alter user [user] quota 10m on [表空间];    --增大表空间
   ~~~

4. 表之间的权限

   ~~~plsql
   grant select on scott.emp to [users];   --新建用户拥有对scott.emp表的查询权限；
   
   grant create any table to [users]； -- 新建用户拥有对scoot建立表的权限。
   
   grant select ,update(sal) on emp to [users]; --新建用户拥有对scott.emp的修改字段，精确到字段。  
   ~~~

5. 系统权限的传递与回收

   * **with admin option**  如果，此时将第一个的权限收回，后面的还会拥有此权限。如：

   ​       --sys赋予权限create session 给tim1,tim1赋予权限create session 给tim2,然后回收tim1的create session权限，tim1肯定就没有此权限了，但是tim2依然有

   ~~~plsql
     grant create session to tim1 with admin option;
     grant create session to tim2;
     revoke create session from tim1;
   ~~~

   * **with grant option**

     如果，此时将第一个的权限收回，后面的也将不会拥有此权限

### 事务

事务具备以下四种特性，简称ACID属性：

1. 原子性：事物是一个完整的动作，事务是一个完整的操作，事务的各步操作是不可分的，要么都执行，要么都不执行。

2. 一致性：一个查询的结果必须与事务在查询开始的状态一致(读不等待写，写不等待读)。

3. 隔离性：对于其他会话来说，未完成的事务是不可见的。

4. 持久性：事务一旦提交完成后，数据库就不可以丢失这个事务结果，数据库通过日志能够保持事务的持久性。oracle主要是通过‘rudo’日志，先记录日志，再写到磁盘上。

事务采用隐性的方式，起始于session的第一条DML语句

查看事务：

~~~plsql
	select * from v$transaction;
~~~

事务结束于：

1. commit 提交 rollback 回滚

2. DDL语句被执行

3. DCL语句被执行

4. 用户推出sqlplus（正常退出是提交，非正常退出是回滚）

5. 机器故障或系统崩溃，回滚

   shutdown immediate ,回滚

### 锁

概述:锁大概分为：共享锁和排他锁

1. 共享锁：排斥其他排它锁，但不排斥其他共享锁；

2. 排它锁：排斥其他锁和共享锁。

锁类型：

1. DML锁(data locks,数据锁)，用于保护数据的完整性。tx(行级锁)，tm(表级锁)，我们日常所使用的dml操作就会产生事务和锁。

2. ~~~plsql
   select * from v$lock;    -- 查看锁信息
   ~~~

3. DDL锁(dictionary locks,数据字典锁),用于保护数据库对象的结构，如表、索引、等的结构定义。

4. SYSTEM锁，保护数据的内部结构。

5. 锁用途：只有事务才会产生锁，保证数据的完整性和正确性。

加锁模式：

​       自动加锁，做DML操作是，如insert,update,delete，以及select …… for update由oracle自动完成加锁。

​       select * from emp where deptno=10 for update;

​       试探要修改数据记录是否被加锁，如下三种形式：

~~~plsql
		select * from emp1 where empno=7782 for update nowait;

        select * from emp1 where empno=7782 for update wait 5;

        select * from emp1 where job=’CLERK’ for update skip locked;
~~~

​        如果这个锁占用的时间太长，我们可以通过管理员杀掉session用户：

​             首先要找到是哪个sid站用了太长时间，查看v$lock表

​              然后根据v$lock表的sid,去$session里面去找到，进行kill操作。

~~~plsql
		  select sid,serial# from v$session where sid =170;

          alter system kill session ‘sid,serial#’;          
~~~

### 表空间

1. 创建表空间

   * 创建临时表空间

     ~~~plsql
     CREATE TEMPORARY TABLESPACE ZCDB_TEMP
     
     TEMPFILE 'D:\oracle\datafile\zcdb_temp.dbf'
     
     size 1024m
     
     autoextend on;
     
     next 50m maxsize 201480m;
     
     extent management local;
     ~~~

   * 创建数据表空间

     ~~~plsql
     CREATE TABLESPACE ZCDB
     
     DATAFILE 'D:\oracle\datafile\zcdb.dbf'
     
     size 1024m
     
     autoextend on;
     
     next 50m maxsize 201480m;
     
     extent management local;
     ~~~

     

   * 给表空间增加数据文件

     ~~~plsql
     alter tablespace zcdb add 
     
     datafile 'D:\oracle\datafile\zcdb2.dbf' 
     
     size 10240m autoextend on;
     
     next 50m maxsize 201480m;
     
     extent management local;
     ~~~

     

2. 创建用户时指定表空间

   ~~~plsql
   CREATE USER ZCDB IDENTIFIED BY ZCDB
   
   DEFAULT TABLESPACE ZCDB
   
   TEMPORARY TABLESPACE ZCDB_TEMP;
   ~~~

3. 查看表空间和使用情况

   ~~~plsql
   SELECT t.tablespace_name,round(SUM(bytes/(1024*1024)),0)ts_size
   
   FROM dba_tablespaces t,dba_data_files d
   
   WHERE t.tablespace_name=d.tablespace_name
   
   GROUP BY t.tablespace_name;
   ~~~

### 同义词 synonym

1. 含义：

   从字面意思来理解就是别名的意思，和视图的功能类似。是一种映射关系。

   私有同义词：一般是普通用户自己建立的同义词，创建同义词需要create synonym权限：

   ~~~plsql
   grant create synonym to scott;
   
   create synonym adc for emp;
   ~~~

   公有同义词：一般是由DBA创建，所有的用户可以使用，创建者需要create public synonym权限

   ~~~plsql
   create public synonym to scott;
   
   create public synonym xyz for emp;
   
   drop public synonym xyz;
   ~~~

2. 要点

   私有同义词是模式对象，一般在自己的模式中使用，如果其他模式使用则必须用模式名前缀限定。

   公有同义词不是模式对象，不能用模式名做前缀。

   私有和公有同义词同名时，如果指向不同的对象，私有同义词优先。

   引用的同义词的对象(表或者视图)被删除了，同义词仍然存在，同视图类似，重新创建该对象名，下次访问同义词是自动编译。

   例如：

   ~~~plsql
     		create synonym wyz from emp1;
   
           drop table emp1;
   
           select * from wyz; -- 已删除表，同义词转换不再有效
   
           flashback table emp1 to before drop;
   
           select * from wyz; -- 利用闪回，同义词再次有效。      
   ~~~

### DBLINK

1. oracle的dblink用于对不同的数据实例或者远程进行连接。

2. 语法：

   ~~~plsql
   CREATE PUBLIC DATABASE LINK LINKNAME
   
   CONNECT TO USERNAME IDENTIFIED BY PASSWD USING
   
   '(DESCRIPTION=
   
   (ADDRESS_LIST=(ADDRESS=(PROTOCOL=TCP)(HOST=)(PORT=)))
   
   (CONNECT_DATA=(SERVICE_NAME=))
   
   )';
   ~~~

3. 使用

   ~~~plsql
   select * from emp@linkname;
   ~~~

### 表的类型

1. 介绍

   表的功能是存储、管理数据的基本单元

2. 类型

   a)      堆表：heap table : 数据存储，行是无序的，进行查询时采用全表扫描

   b)      分区表：建议表>2G才进行分区

   c)      索引组织表(IOL)

   d)      簇表

   e)      临时表

   f)       压缩表

   g)      嵌套表

## 高级sql

1. 复制表

   ~~~sql
   create table<new table> as select * from <exists table>
   ~~~

   要求目标表不存在，因为在插入的时候会自动创建表，并将查询表中的字段数据复制到新建的表中。

   ~~~sql
   insert into table2(f1,f2) select v1,v2... from table1
   ~~~

   要求目标表必须存在

2. merge into的用法

   ~~~plsql
   merge into 表A
   using 与表A产生关联的字段值
   on 进行和表A关联
   	when matched then
   		update set ...
   	when not matched then
   		insert (...) values ...
   ~~~

   * 例如

     ~~~plsql
     merge into products a
     using(select 1717 product_is,’001’ rep_no from dual) b
     on (a.product_id=b.product_id and a.req_no=b.req_no)
     when matched then
     update set product_name=’进行更新啦’, category=’更新的category’
     when not matched then
     insert (product_id,req_no,product_name,category) values(1717,’002’,’新产品’,‘cca’);
     ~~~

3. 递归查询(分层查询),在递归遍历树形结构的时候使用

   ~~~~plsql
   start with （从某个节点id开始）
   connect by prior （子节点id=父节点id）
   形如：
   select * from emp
   start with empno=7369
   connect by prior mgr=empno;（父节点id=子节点id，向上查询，反之，向下查询）
   可以添加where 条件；可以指定多个起始节点的查询；可以进行排序。
   
   ~~~~

## 统计分析函数

1. over

   ~~~plsql
   select empno,sal,ename,deptno,
   sum(sal) over(partition by deptno order by ename) "部门连续求和",
   sum(sal) over(order by deptno,ename) "连续求和",
   sum (sal) over() "薪水总计",
   100*round(sal/sum(sal) over(),5) "份额(%)"
   from emp;
   
   ~~~

   ![1554604097661](C:\Users\kanghailong\AppData\Roaming\Typora\typora-user-images\1554604097661.png)

2. row_number() 函数，进行排名，类似的还有：rank() , dense_rank() 函数

   区别

   ​	row_number:不管分值相不相等没有并列排名的；
   ​	rank():分值相等的时候是有并列排名的，但是如果有两个并列第一，就没有第二名了；
   ​	dense_rank():分值相等的时候是有并列排名的，但是如果有两个并列第一，依然有第二名；

   ~~~plsql
   select ename,sal,deptno,
   row_number() over (partition by deptno order by sal desc) as "排名"
   from emp;
   
   ~~~

   ![1554604153451](C:\Users\kanghailong\AppData\Roaming\Typora\typora-user-images\1554604153451.png)

3. rollup函数

   * 用法：group by 后面接rollup是在纯粹的group by 分组之上再加上earnmonth的汇总统计。

   * 例如

     ~~~plsql
     select earnmoth,area,sum(personincomes)
     from earnings
     group by rollup (earnmoth,area);
     
     ~~~

     

   * ![1554604361527](C:\Users\kanghailong\AppData\Roaming\Typora\typora-user-images\1554604361527.png)

4. cube函数 跟rollup的区别就是，cube(v1,v2),v1和v2中的所有集合配对情况都统计出来。

   ~~~plsql
   select earnmoth,area,sum(personincomes)
   from earnings
   group by cube (earnmoth,area);
   
   ~~~

   ![1554604576090](C:\Users\kanghailong\AppData\Roaming\Typora\typora-user-images\1554604576090.png)

   

5. grouping 函数

   * 用法：带一个参数，如果是本身的结果就返回0，如果是合计的结果就返回1.

   * ~~~plsql
     select ename,
     (case when ((grouping(ename)=1) and (grouping(deptno)=0)) then to_char('部门小计')
           when((grouping(ename)=1) and (grouping(deptno)=1)) then to_char('总计')
           else to_char(deptno) end) as deptno,
     sum(sal)
     from emp
     group by rollup (deptno,ename);
     
     ~~~

   * ![1554604744481](C:\Users\kanghailong\AppData\Roaming\Typora\typora-user-images\1554604744481.png)

   ~~~
   
   ~~~

   

## 优化

### 设计的优化

优化的方案

1. 索引(index)

2. 分区(partition)

3. 物化视图

4. 并行查询

#### 索引

索引分为B树索引和位图索引。下图为B树索引，主要研究下B树索引：

![img](file:///C:/Users/KANGHA~1/AppData/Local/Temp/msohtmlclip1/01/clip_image001.png)



索引概念：

​       B树索引：

​      	 类似于字典查询，最后到leaf block 存的是数据rowid和数据项；

1. 叶块之间使用双向链连接，为了可以范围查询

2. 删除表行时索引叶块也会更新，但只是逻辑更改，并不做物理的删除

3. 索引叶块中不保存表行键值的null信息

位图索引结构

​       离散度比较低的时候，需要用位图索引，离散度指的是重复度。

​       ![img](file:///C:/Users/KANGHA~1/AppData/Local/Temp/msohtmlclip1/01/clip_image002.png)

索引说明：

​       索引是与表相关的一个可选结构，在逻辑上和物理上都独立于表的结构，索引能优化查询但不能优化DML操作，oracle自动维护索引，频繁的DML操作反而会引起大量的索引维护。

​       如果sql语句仅访问被索引的列，那么数据库只需从索引中读取数据，而不用读取表。

​       如果该语句同时还要访问除索引之外的列，那么，数据会使用rowid来查找表中的行，通常，为检索表数据，数据库以交替方式先读取索引块，然后读取响应的表块。

索引的目的：主要是减少io,这是本质，这样才能体现索引的效率。

1. 大表，返回行数<5%

2. 经常使用where字句查询的列

3. 离散度高的列

4. 更新键值代价低

5. 逻辑and,or效率高

6. 查看索引建在那张表/列

   ~~~plsql
   select * from user_indexes;
   select * from user_ind_columns;
   ~~~

索引的使用：

1. 唯一索引，唯一索引指的是键值不重复。

   ~~~plsql
   create unique index empno_idx on emp1(empno)
   ~~~

2. 一般索引：索引键值可以重复

   ~~~plsql
   create index empno_idx on emp1（empno）;
   ~~~

3. 组合索引，绑定了两个或者更多列的索引。

   ~~~plsql
   create index job_deptno_idx on emp1(job,deptno);
   ~~~

4. 反向索引，为了避免平衡索引热块，比如emp表中开头都是‘7’,这样在构建索引的时候何有可能把所有数据都分配到一个块里面，使用反向键值使索引值反向，避免此类问题，使索引树的数据分布均匀。

   ~~~plsql
   create index mgr_idx on emp1(mgr) reverse;
   
   drop index mgr_idx;
   ~~~

5. 函数索引：查询时必须用到这个函数，才会使用到此索引

   ~~~plsql
   create index fun_idx on emp1(lower(ename));
   
   select * from emp1 where lower(ename)=’scott’
   
   drop index fun_idx;
   ~~~

6. 压缩索引(compress)

   ~~~plsql
   create index comp_idx on emp1(sql) compress;
   
   drop index comp_idx;
   ~~~

7. 升序降序索引

   ~~~plsql
   create index deptno_job_idx on emp(deptno desc,job asc);
   
   drop index deptno_job_idx;
   ~~~

索引碎片的问题：

1. 查看执行计划：

   ~~~plsql
   set autotrace traceonly explain;
   ~~~

2. 问题：

   由于对基表做DML操作，导致索引表块的自动更改操作。尤其是基表的delete操作会引起index表的index_entries的逻辑删除，

   注意只有当一个索引块中的全部index_entry都被删除了，才会把这个索引快删除，索引对基本的delete、insert操作都会产生索引碎片的问题。

   oracle建议通过Segment Advisor（段顾问）解决表和索引的碎片问题，如果想自行解决，可以通过查看idnex_stats视图，当以下三种情形之一发生时，

   说明积累的碎片该整理了：

   1. HEIGHT>=4

   2. PCT_USERD<50%

   3. DEL_LF_ROWS/LF_ROWS>0.2

      

   分析索引

   ~~~plsql
   analyze index ind_1 validate structure;
   
   select name,HEIGHT,PCT_USED,DEL_LF_ROWS/LF_ROWS from index_stats;
   
   delete t where rownum<700000;
   
   alter index ind_1 rebuild [online] [talbespace name];
   ~~~

#### 数据库表的设计

1. 数据库表的设计

   * 业务需要学会切分

     逻辑分层（数据库分层）

   * 数据库表结构与拆分(垂直拆分和水平拆分)

     mysql（水平拆分）分片

     分区

     物化视图

     中间表的方法

     设计的方案

   * 对于结构优化的设计

     建立索引

     1. 建立普通索引

     2. 建立规则索引，比如让id索引拥有更多的含义，可以根据业务把常用的字段按照一定的规则拼接到里面。

     3. 复合索引

     4. 数据规则，添加认为必要的扩展字段

     5. 预留字段（用于关联其他业务的）

     6. 做一些合理的冗余

#### 物化视图

1. 概念：

   视图(view)是一种虚表，其目的仅仅是为了方便我们进行综合数据的查询而已，他并不能够帮助我们提高性能。而物化视图，是一种特殊的物理表，“物化”视图是相对于普通视图而言的。

2. 特点：

   物化视图在某种意义上说是一个物理表可以通过user_tables查询出来。

   物化视图是一种段(segment)，所以其有自己的物理存储属性：

   物化视图会占用数据库的磁盘空间，从user_segment查询结果可以得到佐证。

3. 使用

   oracle提供了两种方式，手工刷新和自动刷新，默认为手动刷新。即：ON DEMAND、ON COMMIT：

   二者的区别在于刷新的方法不同，on demand顾名思义，仅在物化视图“需要”刷新的时候才进行刷新，以保证和基表数据的一致性；而on commit是一旦基表有了commit即事务提交，则立即刷新，立刻更新物化视图使得数据和基表一致。

4. 语法：

   ~~~~plsql
   create materialized view mv_name[选项N] as select * from table_name;
   
   [选项1]：BUILD[immediate,deferred] 是否在创建视图时生成数据，默认生成。deferred为不生成数据，需要的时候生成。
   
   [选项2]：REFRESH[FAST,COMPLETE,FORCE,NEVER] fast是增量刷新，或者叫快速刷新；complete为全表刷新；force为如果增量刷新可以使用则使用增量刷新，否则全表刷新；never则是不进行刷新(一般不使用)。
   
   [选项3]：ON[DEMAND,COMMIT] 即手工刷新和提交时刷新。
   
   [选项4]:start with通知数据库完成从主表到本地第一次复制的时间。
   
   [选项5]：next说明了刷新的时间间隔，下一次刷新时间=上一次执行完成时间+时间间隔。
   ~~~~

5. 实例：
   1. 刷新方式force，并且是定时刷新

      ~~~plsql
      CREATE MATERIALIZED VIEW MV_AB   
      REFRESH FORCE 
      ON DEMAND           
      START WITH SYSDATE           
      NEXT SYSDATE+1   
      AS   
      SELECT    
      A.ID,A.NAME,B.CLASSID,B.NAME AS CLSNAME   
      FROM A,B   WHERE   A.CLSID=B.CLSID;   
      ~~~

      **注意**：force,虽然是如果能够增量刷新，就增量否则是全表刷新，但是在这里一定是全表刷新，因为增量刷新需要在基表中加入rowid。

   2. 增量刷新

      ~~~plsql
      CREATE MATERIALIZED VIEW LOG ON A WITH ROWID;   
      CREATE MATERIALIZED VIEW LOG ON B WITH ROWID;       
      CREATE MATERIALIZED VIEW MV_AB   
      REFRESH FAST 
      ON DEMAND           
      START WITH SYSDATE           
      NEXT SYSDATE+1/1440   
      AS   SELECT    
      A.ROWID AS AROWID,B.ROWID AS BROWID,   A.ID,A.NAME,B.CLASSID,B.NAME AS CLSNAME   
      FROM A,B   
      WHERE   A.CLSID=B.CLSID; 
      ~~~

      **注意**：使用fast刷新在查询的时候一定要把rowid显示的查询出来。

#### 分库分表

1. 介绍：

   日常开发中的分库分表的问题，其实都是基于OLTP和OLAP的业务前提，然后对数据进行切分。

   例如垂直、水平切分

2. OLTP和OLAP

   联机事务处理（OLTP）也称为面向交易的处理系统，其基本特征是原始数据可以立即出传送到计算中心进行处理，并在很短的时间内给出处理结果。

   联机分析处理（OLAP）是指通过多维的方式对数据进行分析、查询和报表，可以同数据挖掘工具、统计分析工具配合使用，增强决策分析能力。

3. 垂直切分：

   按照不同的表来分到不同的数据库上，称为数据的垂直切分。

   特点：

   ​        规则简单，实施也方便，尤其是适合业务的耦合性非常低，相互影响很小，业务逻辑非常清晰的系统。这种系统可以很容易做到将不同的业务模块所使用的表拆分到不同的数据库中。根据不同的表来进行切分。对应用程序的影响也更小，拆分规则也简单清晰

4. 水平切分

   根据表中的数据逻辑关系，将同一个表中的数据按照某种条件切分到多台数据库上，称为水平拆分。

   特点：

   ​        相对于垂直拆分，稍微复杂些，因为需要将同一张表中不同数据拆分到不同数据库中，对应用程序而言，拆分规则本身就比按照表名来拆分更复杂，后期的数据维护也会更复杂些。

5. 数据拆分的切点和解决方案
   * 引入分布式事务的问题

     如：业务逻辑非常复杂时，使用soa做通用服务，在server层上做多个切面，配置多个事务。

     如：数据量很大且分析逻辑比较复杂时，可以使用缓冲表缓存表等

     如：要求实时性非常高且数据信息，业务逻辑单一使用第三方数据通信组件，比如消息队列做事务回调服务或者使用zookeeper建立分布式锁进行数据同步，或使用直连的netty进行通信，类似webservice 、restful等请求。

   * 跨节点join的问题，跨节点合并、排序、分页等处理数据问题

     通用方案是把数据组织好以后放到缓存中去，定时或者实时进行同步。如果要求实时性不是特别高，那么也可以使用中间库的手段去解决。

   * 多数据源管理的问题

     使用类似mycat的代理平台，管理过个数据源。

     在每个应用程序模块中配置管理自己所需要的一个或者多个数据源。直接访问各个数据库在模块内完成数据的整合。

#### 分区表

​	主要针对大数据，频繁查询诗句等需求，有了分区表，可以对表进行区间的拆分和组织。

​	提高查询的效率，一般来说，oracle表分区的一个区间数据最好不大于500w条，也就是说500w条数据左右可以划分一个区间。

​	类型：

​      	 range分区

​       	hash分区

​       	list分区

​       	复合分区

​       	间隔分区

​       	system 分区

##### range 分区

range分区就是区域分区，按照定义的区域进行分区。

语法：

~~~plsql
     create table(..)  -- 先创建表

       partition by range (field)(

              partition p1 values less than (value),

              partition p2 values less than (value),

              partition p3 values less than (value)

)

--查看分区的情况：
select * from user_tab_patitions;

--查看分区数据：
select * from table partiotion(p1);
--修改分区：

       添加：alter table tableName add partition p4 values less than (maxvalue);

       删除：alter table tableName drop patition p4;

--注意：更新数据时不可以跨分区操作，会出现错误，需要设置可移动的分区才能进行跨分区查询。

       alter table tableName enable row movement;
~~~

##### Hash分区

1. 简介

   hash 分区实现均匀的负载值分配，增加hash分区可以重新分布数据。

2. 语法

   ~~~plsql
   create table(..)  -- 先创建表
   
   partition by hash(field)(
   
   partition p1,
   
   partition p2,
   
   )
   ~~~

##### list分区

1. 描述

   按照一个字段某几个固定的值进行分区。

2. 语法

   ~~~plsql
      create table(..)  -- 先创建表
   
        partition by list(city)(
   
               partition east values (‘tianjin’,’dalian’),
   
               partition west values (‘xian’),
   
               partition north values (‘haerbin’),
   
               partition other values(default)
   
   );
   ~~~

##### 复合分区

1. 简介

   把范围分区和散列分区相结合或者范围分区和列表分区相结合

2. 语法

   ~~~plsql
   create table(..)  -- 先创建表
   
    partition by range(sno)
   
    subpartition by hash (sname)
   
    subpartitions 4
   
   (
   
   partition p1 values less than (1000),
   
   partition p2 values less than (2000),
   
   partition p3 values less than (maxvalue),
   
   );
   ~~~

##### 间隔分区

1. 简介

   Interval Partitioning 是一种分区自动化的分区。可以指定时间间隔进行分区，是oracle11g的新特性，这个功能在实际工作中经常用到。

   Interval Partitioning实际上是由range分区引申的，最终实现了range分区的自动化。

2. 语法

   ~~~plsql
   create table interval_sale(
   
           sid int,
   
           sdate timestamp
   
   )
   
   partition by range(sdate)
   
   interval (numtoyminterval(1,’MONTH’))
   
   (
   
           partition p1 values less than(TIMESTAMP ‘2014-02-01 00:00:00.00’)
   
   )
   ~~~

   

###### 分区索引

1. 简介：

   分区之后虽然可以提高查询的效率，但也仅仅提高了数据的范围，所以我们在有必要的情况下，需要建立分区索引，进一步提高效率。

   分区索引分为两类：一类是local,一类是global

2. 分类

   local:在每个分区上建立索引。

   global:一种是在全局上建立索引，这种方式分不分区都一样。一般不使用；

   ​        另一种就是自定义数据区间的索引，叫做前缀索引。自定义区域是注意必须要指定maxvalue.

   注意：在分区上建立的索引必须是分区字段列。

3. 语法：

   ~~~plsql
   local方式：
   
           create index idxname on table(field) local;
   
           查看分区索引：select * from user_ind_partitions;
   
   global自定义全局（前缀索引）：
   
           create index idxname on table(field) global
   
           partition by range(field)(
   
           	partition p1 values less than (value),
   
           	partition p2 values less than(maxvalue)
   
   		);
   
   global 全局索引方式语法： create index idxname on table (field) global;
   ~~~

   

### sql的优化

#### 避免索引失效的情况

1. IS NULL 与 IS NOT NULL

   只要这些列中有一列含有null，该列就会从索引中排除。也就是说如果某列存在空值，即使对该列建索引也不会提高性能。

2.  联接列   ||

   ~~~plsql
    select * from employss where first_name||''||last_name ='Beill Cliton';
    -- 改写
    select * from employss where first_name ='Beill' and last_name ='Cliton';
    
    -- 第一种写法会让索引失效，第二种则不会
   ~~~

3. 带通配符(%)的like语句

   ~~~plsql
   -- 出现在词首会引起索引失效，词尾则不会
   select * from employee where last_name like '%cliton%';
   select * from employee where last_name like 'c%';
   ~~~

4. Order by语句

   仔细检查order by语句以找出非索引项或者表达式,应绝对避免在order by子句中使用表达式。

5. NOT  <>

   ~~~plsql
   select * from employee where salary <> 3000;
   --改写为
   select * from employee where salary<3000 or salary>3000;
   
   ~~~

6. 是否在条件列上施加了函数、运算

   ~~~plsql
   select * from t1 where trunc(id)=xxx; 
   
   select * from t1 where id-3=xxx;
   --假设在ID列上存在索引，这种写法会导致索引不可用。
   --可尝试做等价改写，避免在ID列上施加函数，而在值上施加函数或运算。
   ~~~

   

#### plsql优化

1. 使用DECODE或CASE WHEN 替代IF THEN,DECODE和CASE WHEN可以实现批理数据的处理.。

   ~~~plsql
    IF V_NAME = 'AS_CC_01_01' THEN
         V_JNAME := 'AVG_MOB_RC';
       ELSIF V_NAME = 'AS_CC_01_02' THEN
         V_JNAME := 'MAX_DELQ_3M_TOTAL';
       ELSIF V_NAME = 'AS_CC_01_03' THEN
         V_JNAME := 'MONTH_SNC_LAST_CA_QUERY';
       ELSIF V_NAME = 'AS_CC_01_04' THEN
         V_JNAME := 'NET_UTL_OPEN_CC';
       ELSIF V_NAME = 'AS_CC_01_05' THEN
         V_JNAME := 'NO_NORMAL_OPEN_3M_CC';
       ELSIF V_NAME = 'AS_CC_01_06' THEN
         V_JNAME := 'NO_QUERY_3M_CA';
       ELSIF V_NAME = 'AS_CC_01_07' THEN
         V_JNAME := 'P_DELQ_24M_RC';
       ELSIF V_NAME = 'AS_CC_01_08' THEN
         V_JNAME := 'P_NEVERDELQ_6M_CC';
       ELSIF V_NAME = 'AS_CC_01_09' THEN
         V_JNAME := 'P_UTL30_CC';
       ELSIF V_NAME = 'AS_CC_01_10' THEN
         V_JNAME := 'SD_C1_EDUC_CD';
       ELSIF V_NAME = 'AS_CC_01_11' THEN
         V_JNAME := 'SD_C1_GENDER_CD';
       ELSE
         V_JNAME ：=‘’
         
         
     -- 改写
    DECODE(CT.NAME,
                                    'AS_CC_01_01',
                                    'AVG_MOB_RC',
                                    'AS_CC_01_02',
                                    'MAX_DELQ_3M_TOTAL',
                                    'AS_CC_01_03',
                                    'MONTH_SNC_LAST_CA_QUERY',
                                    'AS_CC_01_04',
                                    'NET_UTL_OPEN_CC',
                                    'AS_CC_01_05',
                                    'NO_NORMAL_OPEN_3M_CC',
                                    'AS_CC_01_06',
                                    'NO_QUERY_3M_CA',
                                    'AS_CC_01_07',
                                    'P_DELQ_24M_RC',
                                    'AS_CC_01_08',
                                    'P_NEVERDELQ_6M_CC',
                                    'AS_CC_01_09',
                                    'P_UTL30_CC',
                                    'AS_CC_01_10',
                                    'SD_C1_EDUC_CD',
                                    'AS_CC_01_11',
                                    'SD_C1_GENDER_CD')  
   ~~~

   

2. 查询语句使用UNION

   UNION隐含有排序去重处理。除非必要，应使用UNION ALL代替。

3. 统计信息是否过旧

   统计信息会影响执行计划的确定，过旧的统计信息，可能会导致低效执行计划的产生。

4. 当返回的数据量较大时，慎用自定义函数

5. 慎用游标

   批量集合处理的效率远高于循环单条处理的效果。

6. 当返回的数据量较大时，慎用标量子查询