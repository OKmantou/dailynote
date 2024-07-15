# Oracle基础知识

## 基础概念

### **数据库**

从数据库本身的含义来讲，指的是存储数据的数据文件，会落到磁盘上。但是广义上讲，数据库包含了数据文件、以及数据库管理系统。

Oracle在创建时会要求创建一个启动数据库，也称为全局数据库，还要求写入一个全局数据库名。这个名字会对应到之后创建的各种控制文件以及数据库表中。

启动数据库是数据库系统的入口，会内置一些高级权限的用户如 SYS、SYSTEM等。利用这些高权限用户可以在数据库实例中创建表空间、用户、表。

```
select name from v$database;
# 查看当前数据库名
```

#### 物理存储结构

数据库的物理结构是一堆文件的集合。

**数据文件**

存储数据库数据的文件，后缀是.dbf。数据文件主要包括：表、索引数据、数据字典、存储过程、临时文件等。

与数据文件相关的表和视图主要包括：dba_data_files, dba_temp_files, v$datafile, v$tempfile等。

```sql
select file#, name from v$datafile;

# 查询数据文件详细信息
select file_name, tablespace_name, bytes/(1024*1024)MB from dba_data_files;

# 重置数据文件大小
alter database datafile '/u01/oracle/datafile.dbf' resize 1000MB;
```

**控制文件**

用于记录和维护数据库的全局物理结构，以.ctl结尾。一个数据库至少包含一个控制文件（可以有多副本），控制文件内主要包含：数据库名、标识符、数据库创建时间、表空间名称、联机重做日志位置、检查点等。

控制文件相关视图：v$controlfile, v$parameter等。

```sql
select name from v$controlfile;
```

**日志文件**

用于记录数据库事务的处理过程，后缀为.log。用户对数据库的修改先在高速缓冲区中，通过将重做日志写入日志缓冲区，到达一定条件后，由DBWn进程将缓冲区中的内容写入磁盘，重做日志缓冲区的内容由LGWR写入重做日志文件。为了保证日志文件的可用性，采用日志组。

日志文件相关视图：v$log, v$logfile等

```sql
select group#, sequence#, members, archived, status, first_time from v$log;
```

**参数文件**

主要包括两种：

- 静态参数文件(pfile)：文件为正文文件，文件名一般为initSID.ora
- 动态服务器参数文件(spfile)：二进制文件，spfileSID.ora

```sql
show parameter pfile;
```

#### 逻辑存储结构

逻辑结构存储在数据库的数据字典中。数据库的逻辑存储结构分为：

- 块：数据库最小的IO单元
- 区：由若干个块组成，是数据库最小的分配内存单元
- 段：由若干区组成，存储相同类型的数据。一般情况下一个数据库有一个段
- 表空间：由若干段组成，是最大的存储逻辑单元

![Description of Figure 12-1 follows](https://docs.oracle.com/en/database/oracle/oracle-database/19/cncpt/img/cncpt227.gif)

### 数据库实例

数据库实例是访问数据库必须的：<u>计算机内存+辅助处理后台</u>进程。进程使用的内存也叫做SGA。数据库实例是数据库的<u>运行时环境</u>，负责管理数据库的物理和逻辑资源，以及处理用户的请求。

实例名是用于响应某个数据库操作的数据库管理系统名称，叫做系统标识（System Identifier，SID），用于在网络中表示数据库实例，由参数instance_name决定。一个数据库可以有多个实例。

```sql
select instance_name from v$instance;
# 查看当前数据实例名
```

开发程序连接时使用（jdbc:oracle:thin:@localhost:1521:**orcl**），其中orcl就是实例名。

<img src="https://docs.oracle.com/en/database/oracle/oracle-database/19/cncpt/img/cncpt223.gif" alt="img" style="zoom:67%;" />

**SGA**

所有用户都可以访问的共享内存区

 **PGA**

针对特定服务器进程访问的内存区

#### 内存区

**SGA**

在上图的实例中，SGA是一个全局共享的内存区（show sga;查看各区大小），主要包含了：

- shared pool：缓存数据字典、已经分析过的SQL语句等
- fixed SGA：记录实例固定的参数
- database buffer cache：存放读取的数据库文件的数据块副本，采用LRU
- redo log buffer：记录对数据库的更改，重做日志写到磁盘之前首先被放入这个buffer

**PGA**

program global area是一个非共享的内存区，是用户进程链接到实例后分配的缓存。

#### 数据库进程

**用户进程**（会话）

用户在使用客户端连接时，在数据库实例主机上创建出来的，用于相应用户请求的进程（只是相应，不包括分析和执行），可以通过v$session来查看。

**服务器进程**

主要用来分析和执行SQL语句，当所需数据不在SGA中时，就需要从磁盘数据复制到SGA中。

**后台进程**

后台进程是实例启动时就创建的，用来维护实例运行时的状态。主要包括：

- DBWn：数据库写进程。将数据告诉缓冲区中的脏缓冲区中的数据写到数据文件中
- CKPT：：检查点进程
- LGWR：日志写进程
- PMON：监控服务器进程，当某个服务器进程奔溃后，PMON进程负责回滚用户当前事务、释放用户所占用的锁、释放其他资源
- SMON：监控进程，实例恢复

### 表空间

**基本概念**

表空间是表、索引和其他数据库对象的逻辑存储单元。表空间的目的是为了管理和组织数据。一个数据库可以有多个实例，一个实例可以有多个表空间，一个表空间有多张表。一个实例内，可以看到不同的表空间；实例之间一般不能看到表空间的数据。

一个数据库安装完成后会创建出<u>五个基本的表空间</u>。其中SYSTEM和SYSAUX为系统表空间。为了便于管理和提高运行效率，可以使用附加表空间来划分用户和应用程序，例如USER供用户使用，UNDOTBS由回滚段使用。

```sql
select tablespace_name, initail_extent, extent_management, allocation_type from dba_tablespaces;
```

| 表空间类型 | 用途说明                                          |
| ---------- | ------------------------------------------------- |
| SYSTEM     | 存放SYS和SYSTEM用户的数据库数据字典的相关表、视图 |
| SYSAUX     | 附加的数据库组件                                  |
| TEMP       | 排序时使用的过度数据                              |
| UNDOTBS    | 事务回滚数据                                      |
| USERS      | 存放用户私有数据                                  |

表空间管理方式

1. 数据字典：



**与表空间相关的视图和表**

- v$tablespaces：从控制文件获取的表空间
- dba_tablespaces
- v$datafile
- dba_data_files
- dba_free_space

```sql
#
select * from v$tablespace;

#表空间数据文件
select file_name, blocks, tablespace_name from dba_data_files;

#查询表空间剩余大小
select tablespace_name, sum(bytes)free_spaces from dba_free_space group by tablespace_name;
```



```
Create TableSpace 表空间 DataFile 表空间数据文件路径 Size 表空间初始大小 Autoextend on;
```

![Description of Figure 11-4 follows](https://docs.oracle.com/en/database/oracle/oracle-database/19/cncpt/img/cncpt037.gif)

### **用户**

除了在创建数据库时，默认生成的SYS和SYSTEM用户外，还需要创建其他权限的用户用于连接数据库实例，并且为用户指定表空间。

创建新用户：

```
CREATE USER 用户名 IDENTIFIED BY 密码 DEFAULT TABLESPACE 表空间(默认USERS) TEMPORARY TABLESPACE 临时表空间(默认TEMP)
```

为用户分配权限：

```
GRANT CONNECT TO utest; GRANT RESOURCE TO utest; # 赋予连接权
GRANT dba TO utest;	#dba为最高级权限，可以创建数据库，表等
```

### 表

schema是一个抽象概念，包括了tables、views、sequences、stored procedures、indexes等。一个用户一般对应一个schema，用户名与schema名相同。Oracle数据库不能创建新的schema，只能通过创建用户解决。

当访问scott用户下的emp表，使用 `select * from emp`，系统会补充完整的sql为 `select * from scott.emp`

**用户和实例的关系**

用户连接数据库实例：用户通过数据库客户端（sqlplus、sql developer等）通过网络连接到数据库实例。

用户权限与实例的权限管理：实例负责管理用户的权限，用户在数据库中被分配了不同的角色和权限。

### 数据字典

**静态数据字典**

静态数据字典由是数据库的元数据的静态快照，它记录了数据库对象（如表、索引、视图等）的结构和定义，以及其他数据库级别的信息。

静态数据字典的内容在数据库启动时被加载到内存中，并在数据库运行过程中保持不变，直到数据库重新启动。

数据字典视图是预先定义在数据字典上的表或视图，分为三类：

- dba_视图：数据库中的所有视图，只有dba和有对应权限的用户可以访问
- all_视图：当前用户可以访问的视图
- user_视图：当前用户拥有的视图。常用的视图包括：user_users, user_tablespaces, user_tables, user_views, user_objects

**动态性能视图**

动态数据字典是实时反映数据库当前状态的视图集合，它存储了数据库运行时的信息，例如活动会话、锁定状态、性能统计等。以v$(针对某个实例)，gv$(多实例)， x$开头

常用的性能视图：v$controlfile，v$database，v$datafile，v$instance，v$parameter，v$sesion，v$sga

## 权限管理

Oracle数据库中创建的新用户没有任何权限，需要由拥有授权权的用户来授权。

### 权限

Oracle中将权限分为两类：系统权限和对象权限。

**系统权限**

可以执行在数据库层面，或者某一类对象的动作。例如create user，create session

常见的系统权限有：

- create session，create sequence，create table，drop table，create user

授权、回收语句为：

```sql
grant privilege[,privilege...] to user[,user|role,public];
# public 所有用户

revoke {privilege|role} from {user|role|public}
```

与权限相关的表：`dba_sys_privs`, `user_sys_privs`

**对象权限**

对某一个对象的操作。例如查表。对象的拥有者有所有权限。

授权、回收语句：

```sql
grant [privilege] on [object] to [user]
revoke [privilege] on [object]
```

**基础操作**

```sql
------dba用户查询-----------
# 查询用户的系统权限
select privilege from dba_sys_privs where grantee = '';

# 查询用户角色权限
select granted_role from dba_role_privs where grantee = '';

# 查询用户对象权限
select privilege, table_name from dba_tab_privs where grantee = '';

-------普通用户查询--------------
# 将dba_ 换成 user_，并且根据表字段名更改select和where中的内容
```



### 用户

**概念**

用户用于接入数据库，并访问数据库中的内容。一个合法的用户必须提供username、authentication credential。credential包括密码或者PKI。

- sys：系统最高权力用户
- system：本地管理员，权利次高

**基础操作**

```
# 查看当前登录的用户
show users;

# 创建用户

# 删除用户

# 修改用户密码、账户状态


```



### 角色

role是一组权限的集合，可以被授予一个用户或者另一个role。role可以用来管理一个database application或者一组用户的权限。

Oracle数据库提供了三种标准角色：

- CONNECT role

  临时用户，只需要连接，并赋予访问权限的用户

- RESOURCES role

  更可靠的用户，可以创建自己的资源

- DBA role

  所有系统权限，包括：授权、撤销

  ```sql
  grant connect, resource to uers1;
  revoke connect, resource from user1;
  ```

拥有创建角色权限的用户可以创建新的角色：

```sql
create role stu;

# 授予stu角色对class表select权利
grant select on class to stu;

drop role stu;
```

## 网络配置

Oracle数据库的网络组件（Oracle Net Services）包括三个部分：

- Oracle Net：驻留在server和client的，用于建立连接的部分
- Oracle Net Listener：在数据库主机节点上，用于监听client发来的建立连接请求，可以根据具体的后端负载进行调整。
- 网络配置工具

### 监听器

**Service Names**

client在和service建立连接时，可以通过命名服务连接到后端对应的数据库，同时不同的域名还可以对应同一个数据库。

**Service Registration**

动态地将数据库实例注册到listener

**SCAN**

https://www.cnblogs.com/tinazzz/p/7082877.html

single client access name，客户端可以通过SCAN ip负载均衡地连接到RAC数据库，并且SCAN的切换完全由服务端负责。

```
# 查看当前SCAN VIP 是否运行，运行在哪个节点
srvctl status scan

```

**各种IP**

- public ip：RAC节点真实的ip，一般客户端不通过该ip连接
- private ip：私网ip，用于心跳和数据同步。如果该私网不通，可能会导致脑裂。通常使用网卡绑定做冗余
- virtual ip：每个节点都会有一个vip，该vip会绑定到public ip的网卡上，当节点故障，该vip会漂移到其他节点上，待节点恢复之后会重新漂移回节点。

**监听**

- local listenner：本地监听负责监听public ip和vip。当实例启动时向监听注册。
- scan listenner：该监听跟随scan ip漂移而漂移。该监听受PMON进程管理

### 配置文件

**listener.ora**

${Oracle_home}/network/admin下，包含监听端口、协议和进程。

在一个RAC架构中，每个节点的文件都是相同的。

```
LISTENER_SCAN1=(DESCRIPTION=(ADDRESS_LIST=(ADDRESS=(PROTOCOL=IPC)(KEY=LISTENER_
SCAN1))))               # line added by Agent
LISTENER_NODE1=(DESCRIPTION=(ADDRESS_LIST=(ADDRESS=(PROTOCOL=IPC) (KEY=LISTENER))))
          # line added by Agent
# listener.ora.mycluster Network Configuration File:
/u01/app/oracle/product/18.0.0/dbhome_1/network/admin/listener.ora.mycluster
# Generated by Oracle configuration tools.
 
LISTENER_NODE1 =
  (DESCRIPTION_LIST =
    (DESCRIPTION =
      (ADDRESS = (PROTOCOL = IPC)(KEY = EXTPROC1521))
    )
  )
 
ENABLE_GLOBAL_DYNAMIC_ENDPOINT_LISTENER_NODE1=ON		# line added by Agent
ENABLE_GLOBAL_DYNAMIC_ENDPOINT_LISTENER_SCAN2=ON		# line added by Agent
ENABLE_GLOBAL_DYNAMIC_ENDPOINT_LISTENER_SCAN1=ON		# line added by Agent

```

**sqlnet.ora**

主要存储通信参数，控制server和client的行为

**tnsnames.ora**

/u01/app/product/19.0.0/db_1/network/admin/tnsnames.ora

存储数据库实例的地址和网络名映射文件。

## redo、undo表空间

### undo表空间

用于存储事务回滚信息的表空间，用于保持数据的一致性和完整性。如果事务需要回滚，则会读取undo中的信息来将数据库状态恢复到事务开始之前的状态。

- 闪回查询

  查询和恢复数据库在过去某个时间点的数据状态，而不需要使用备份和恢复方法。闪回查询的可用性和精度取决于数据库管理系统的支持和配置。

  使用场景主要有：数据还原、误操作、数据审计

  闪回查询技术：闪回查询语句、闪回表、闪回日志


## 数据库云

### CDB

container database是多租户架构的顶层容器，一个数据库实例只能包含一个CDB，在CDB中可以包含多个独立的PDB。CDB由DBA创建，提供共享的系统级资源，例如内存和后台进程。

主要作用包括：

1. 管理和监控多个PDB，每个PDB都是一个完成的数据库，拥有自己的数据文件，表空间、用户和对象。
2. 资源共享和优化，CDB可以共享系统级资源给PDB，以提高资源利用率，并且可以动态分配资源。
3. 简化管理和降低成本，将多个独立数据库集中在一个CDB中

### PDB

pluggable database是多租户架构中的独立数据库单元。每个PDB都是一个完整的数据库，包括独立的数据文件、用户、系统表空间和对象。PDB是CDB中的逻辑数据库，

主要特点包括：

1. 独立性：每个PDB互相隔离，可以独立备份、恢复、升级和对性能调整
2. 共享：每个PDB会共享CDB的系统级资源，例如内存和后台进程
3. 管理：每个PDB由独立的管理员维护。通过CDB可以集中管理和维护所有PDB







## QA

1. 数据库架构

   RAC，DG，2+2，1+1

2. 数据库访问与域名

3. 多实例数据的数据表空间是否独立

4. 数据库搭建
