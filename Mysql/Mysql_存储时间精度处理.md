# 一、mysql 数据库日期类型

   名称  |  描述   | 字节数 |  格式 | 取值范围 | 零值|   
   -------|-------|------- 
   | year  | 表示时间   | 1  | yyyy|1901-2155| 0000 |   
   | date  | 表示日期|4|yyyy-mm-dd|1000-01-01~9999-12-31|0000-00-00|   
   | time  | 表示一天中的时间|3|HH:mm:ss|-838:59:59~838:59:59|00:00:00|   
   | dateTime  | 表示日期|8|yyyy-mm-dd|1000-01-01 00:00:00~9999-12-31 23:59:59|0000-00-00 00:00:00|     
   | timestamp  | 表示日期|4|yyyy-mm-dd|19700101080001~20380119111407|00000000000000|
# 二、常用的两种
### 1、 timestamp   
  1. 5.7版本之前,不支持一张表中存在两个默认值为CURRENT_TIMESTAMP 的timestamp 字段，   
       也就是说如果有一个create_time 和 update_time ，两个字段只能有一个默认值为 CURRENT_TIMESTAMP，   
       而且需要在第一个timestamp类型字段设置，值得注意的是如果当前的timestamp字段为非空的话默认是根据当时时间戳更新，   
       如果当前字段是创建时间的话，每次修改数据可能导致创建时间随之更新，需注意。
  2. 5.7版本到之后的版本，一张表中timestamp字段都可以使用默认值CURRENT_TIMESTAMP，都可以根据当前时间戳更新。   
       但是，在5.7版本中，timestamp 类型的字段默认是不允许设置为零日期的。（oracle 数据库迁移到mysql5.7时遇到的问题）   
       如果有相应的业务需求，需要修改mysql的配置文件。   
       
 > 修改步骤：
   >> 1. 执行命令:select @@sql_mode;
   >> 2. 可以看出有两个字段 ：NO_ZERO_IN_DATE 和 NO_ZERO_DATE 两个值，限制字段不能为零日期;
   >> 3. 修改配置文件my.cnf,去掉这两个值 ，并且添加一行:   
            sql_mode=NO_AUTO_VALUE_ON_ZERO,STRICT_TRANS_TABLES,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION,PIPES_AS_CONCAT,ANSI_QUOTES。
            
  3. mysql 的timestamp 值自动从当前时区转换到utc时区存储，并且自动从utc时区转换为当前系统时区检索返回。   
  ### 2、datetime
  1. 在mysql5.7之后，datetime字段也可以指定默认值，并且格式和timestamp一样。
# 三、数据库时区
### 1、 查看数据库时区   
        show variables like '%time_zone';
### 2、 system_time_zone   
- 系统默认时区
### 3、 time_zone
- 当前使用的时区，默认使用的是系统时区
### 4、 修改时区
1. 通过my.conf 文件修改
 > 在 [mysqld] 之下加   
    default-time-zone = '+8:00'
2. 通过命令行修改
 > set time_zone='+8:00';
### 5、验证时区修改是否正确
- select now();通过结果对比。
# 四、存储精度问题
1. 在mysql5.6.4 之前，时间只能精确保存到秒;5.6.4~5.7 之间，精确到分数秒;5.7到之后，精确到微妙。
2. 在数据库存储中，如果字段类型是datetime或timestamp，长度可以设为3，增加精确度,最后可以设为6，不过一般三位就够用了。    
   值得注意的是，如果要加长度，如果数据库版本过低，需要去掉 （根据当前时间戳更新），不然会报错。
    