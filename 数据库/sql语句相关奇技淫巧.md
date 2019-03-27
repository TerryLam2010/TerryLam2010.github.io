# SQL 语句相关的奇技淫巧

## 1.根据表 生成sql 语句

模板 

```
SELECT TABLE_NAME,CONCAT('ALTER TABLE  ',TABLE_NAME,' DEFAULT CHARACTER SET ',a.DEFAULT_CHARACTER_SET_NAME,' COLLATE ',a.DEFAULT_COLLATION_NAME,';') executeSQL FROM information_schema.SCHEMATA a,information_schema.TABLES b
WHERE a.SCHEMA_NAME=b.TABLE_SCHEMA
AND a.DEFAULT_COLLATION_NAME!=b.TABLE_COLLATION
AND b.TABLE_SCHEMA='数据库名' 
```


例子

```
select table_name,
CONCAT('ALTER TABLE  ',table_name,' ADD INDEX idx_user_id(user_id);') 
executeSQL from information_schema.tables
where table_schema='pcbaby_mmban' and table_name like '%mmban_user_integral_record_%';

```



## 2.导入数据

从b表导入a表
// 设置第几行开始

```
SET @rownum= 2 ;
```

// 与上面一起执行  正常id

```
(SELECT @rownum := @rownum +1)
```

例子：

```
-- 一同运行
SET @rownum= 2 ;

insert into mmban_hospital(hospital_id,hospital_name,state_name,city_name,district_name,hospital_address,hospital_intro,phone)
select (SELECT @rownum := @rownum +1) as hospital_id,hospital_name,state_name,city_name,district_name,hospital_address,hospital_intro,phone from test_hospital

```

3. 行数据 并成列查询插入

   ```
   SET @rownum= 2 ;
   insert into mmban_doctor
   SELECT  (SELECT @rownum := @rownum +1), hospital_id,
          doctor_name1,doctor_pic1,doctor_title1,department1,1,NOW(),0,'',NOW(),0,''
   FROM test_hospital1
   where doctor_name1 is not null
   UNION ALL
   SELECT  (SELECT @rownum := @rownum +1), hospital_id,
          doctor_name2,doctor_pic2,doctor_title2,department2,1,NOW(),0,'',NOW(),0,''
   FROM test_hospital1
   where doctor_name2 is not null
   UNION ALL
   SELECT   (SELECT @rownum := @rownum +1),hospital_id,
          doctor_name3,doctor_pic3,doctor_title3,department3,1,NOW(),0,'',NOW(),0,''
   FROM test_hospital1
   where doctor_name3 is not null
   ```

   