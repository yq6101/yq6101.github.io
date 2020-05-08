---
title: Oracle表空间扩容
date: 2020-05-08 16:19:14
tags:
- Oracle
- 数据库
categories: 技术栈
---



## 1.表空间查询

```sql
SELECT TABLESPACE_NAME "表空间",
       To_char(Round(BYTES / 1024, 2), '99990.00')
       || ''           "实有",
       To_char(Round(FREE / 1024, 2), '99990.00')
       || 'G'          "现有",
       To_char(Round(( BYTES - FREE ) / 1024, 2), '99990.00')
       || 'G'          "使用",
       To_char(Round(10000 * USED / BYTES) / 100, '99990.00')
       || '%'          "比例"
FROM   (SELECT A.TABLESPACE_NAME                             TABLESPACE_NAME,
               Floor(A.BYTES / ( 1024 * 1024 ))              BYTES,
               Floor(B.FREE / ( 1024 * 1024 ))               FREE,
               Floor(( A.BYTES - B.FREE ) / ( 1024 * 1024 )) USED
        FROM   (SELECT TABLESPACE_NAME TABLESPACE_NAME,
                       Sum(BYTES)      BYTES
                FROM   DBA_DATA_FILES
                GROUP  BY TABLESPACE_NAME) A,
               (SELECT TABLESPACE_NAME TABLESPACE_NAME,
                       Sum(BYTES)      FREE
                FROM   DBA_FREE_SPACE
                GROUP  BY TABLESPACE_NAME) B
        WHERE  A.TABLESPACE_NAME = B.TABLESPACE_NAME)
--WHERE TABLESPACE_NAME LIKE 'CDR%' --这一句用于指定表空间名称
ORDER  BY Floor(10000 * USED / BYTES) DESC;
```



## 2.查询数据文件路径以及对应表空间

```sql
select b.file_id　　文件ID,
　　b.tablespace_name　　表空间,
　　b.file_name　　　　　物理文件名,
　　b.bytes　　　　　　　总字节数,
　　(b.bytes-sum(nvl(a.bytes,0)))　　　已使用,
　　sum(nvl(a.bytes,0))　　　　　　　　剩余,
　　sum(nvl(a.bytes,0))/(b.bytes)*100　剩余百分比
　　from dba_free_space a,dba_data_files b
　　where a.file_id=b.file_id
　　group by b.tablespace_name,b.file_name,b.file_id,b.bytes
　　order by b.tablespace_name
```



## 3.三种扩容方式

### 方式1：手工改变已存在数据文件的大小

```sql
ALTER TABLESPACE *** ADD DATAFILE
'\****\****.DBF' SIZE 20480M;
```

### 方式2：允许已存在的数据文件自动增长

```sql
ALTER DATABASE DATAFILE '\****\****.DBF'
AUTOEXTEND ON NEXT 100M MAXSIZE 20480M; 
```

### **方式3：增加数据文件**

```sql
ALTER TABLESPACE *** ADD DATAFILE
'\****\****.DBF' 
size 7167M autoextend on ;
```