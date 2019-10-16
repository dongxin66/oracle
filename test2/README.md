# **test2 ：用户及权限管理**
## 第1步：
### 以system登录到pdborcl，创建角色con_res_view和自己的用户new_user_zj，并授权和分配空间：
#### 代码：

 - CREATE ROLE con_res_view;
GRANT connect,resource,CREATE VIEW TO con_res_view;
CREATE USER new_user_dongx IDENTIFIED BY 666 DEFAULT TABLESPACE users TEMPORARY TABLESPACE temp;
ALTER USER new_user_dongx QUOTA 50M ON users;
GRANT con_res_view TO new_user_dongx;
exit
![第一步](https://img-blog.csdnimg.cn/20191015114213876.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0dyZWVuQmlyZFo=,size_16,color_FFFFFF,t_70)
## 第2步：新用户new_user_zj连接到pdborcl，创建表mytable和视图myview，插入数据，最后将myview的SELECT对象权限授予hr用户。
#### 代码：

 - CREATE TABLE mytable（id number，name varchar（50））; INSERT INTO
   mytable（id，name）VALUES（1，'董'）; INSERT INTO
   mytable（id，name）VALUES（1，'鑫'）;
    CREATE VTEW myview AS SELECT name PROM mytable; 
    SELECT FROM myview; 
    GRANT SELECT ON myview To hr;
    desc  mytable; 
    select   *  fron mytable; 
    drop table mytable;

![第二步](https://img-blog.csdnimg.cn/20191015114418508.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0dyZWVuQmlyZFo=,size_16,color_FFFFFF,t_70)
## 第3步：用户hr连接到pdborcl，查询new_user_zj授予它的视图myview。
#### 代码：

 - SELECT * FROM new_user_zj.myview;
![第三步](https://img-blog.csdnimg.cn/20191015114532937.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0dyZWVuQmlyZFo=,size_16,color_FFFFFF,t_70)
## 查看数据库的使用情况
#### 代码

 - SELECT tablespace_name,FILE_NAME,BYTES/1024/1024 MB,MAXBYTES/1024/1024 MAX_MB,autoextensible FROM 
dba_data_files WHERE tablespace_name='USERS';
SELECT a.tablespace_name "表空间名",Total/1024/1024 "大小MB",
free/1024/1024 "剩余MB",( total - free )/1024/1024 "使用MB",
 Round(( total - free )/ total,4)* 100 "使用率%"
 from (SELECT tablespace_name,Sum(bytes)free
        FROM   dba_free_space group  BY tablespace_name)a,
       (SELECT tablespace_name,Sum(bytes)total FROM dba_data_files
        group  BY tablespace_name)b
 where  a.tablespace_name = b.tablespace_name;
![查看数据库使用情况](https://img-blog.csdnimg.cn/2019101511532922.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0dyZWVuQmlyZFo=,size_16,color_FFFFFF,t_70)
### 分析：

 - 随着用户往表中插入数据，表空间的磁盘使用量会增加。 表空间中存储了很多相同名称的表mytable和视图myview，但分别属性于不同的用户，不会引起混淆

