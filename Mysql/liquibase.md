# liquibase
## 背景
 - 数据库变更管理有两种方式，状态和迁移方法；
 - 第一种，基于状态（或声明性），在其中定义数据库的所需状态。
   - 可以将目标环境和定义的所需状态进行比较的工具用于生成允许目标环境与声明的状态匹配的迁移脚本。
 - 第二种，基于迁移（或命令式）的方法，其中描述了用于更改数据库状态的特定迁移。   
   - 使用一种能够显式跟踪和排序单个迁移并部署尚未部署到目标环境的迁移的工具，以使目标数据库得到正确迁移。
 - 从根本上来说，Liquibase 是基于迁移的解决方案，其中的diff功能仅用于协助新项目的启动和健全性检查，以确保数据库迁移已正确应用。
## 定义   
 - 用于数据库重构和迁移的开源工具;
 - 用于跟踪、管理和应用数据库的变化;
 - 通过日志文件的形式记录数据库的变更，执行日志文件中的修改，将数据库更新或回滚到一致的状态;
 - 它将所有数据库的变化（包括结构和数据）都保存在XML文件中，便于版本控制;
 - 始终提供一种与数据库无关的方式来交付快速、安全、可重复的数据库部署。      
## 工作原理
 - 核心是一种简单的机制来跟踪、版本化和部署更改:
   1. 使用变更日志(变更分类账)以特定顺序显式列出数据库变更。
   2. 使用一个跟踪表，该表驻留在每个数据库上，并且跟踪已在变更日志中部署了哪些变更集。
   3. 借助 分类账和跟踪表，Liquibase能够：   
    - 跟踪和版本化数据库更改–用户确切地知道哪些更改已部署到数据库以及哪些更改尚未部署。
    - 部署更改–具体来说，通过将分类帐中的内容与跟踪表中的内容进行比较，Liquibase能够仅部署以前尚未部署到数据库的更改。
   4. 如果Liquibase所作用的数据库上没有跟踪表，则Liquibase将创建一个跟踪表。
   5. Liquibase具有高级功能，例如上下文，标签和前提条件，可精确控制何时以及在何处部署changeSet。
## 特性
 1. 跟踪所有提议的数据库更改，包括需要部署的特定顺序，提议/编写更改的人，并记录更改的目录（作为注释）。
 2. 明确回答是否已将数据库更改部署到数据库。有效地，Liquibase能够"版本化"每个数据库。  
 3. 确定性地将更改部署到数据库，包括将数据库升级到特定的版本。
 4. 防止用户修改已经部署到数据库的更改，并要求他们故意重做已部署的更改或前滚。
## 大白话特点
 1. 几乎支持所有主流的数据库，如Mysql、Oracle、PostgreSql 和 Db2等;
 2. 支持多开发者的协作维护；
 3. 其日志文件支持多种格式，比如Sql、xml、yaml和json;
 4. 支持多种运行方式，比如命令行、Maven和Gradle等。
## 常用命令
1. 创建表
     ```
         <changeSet id="init-schema" author="17835059864@163.com" >
           <comment>init schema</comment>
           <createTable tableName="user" remarks="用户表">
             <column name="id" type="bigint" autoIncrement="${autoIncrement}" remarks="主键">
               <constraints primaryKey="true" nullable="false"/>
             </column>
             <column name="name" type="varchar(255)" remarks="名称">
               <constraints  nullable="false"/>
             </column>
             <column name="password" type="varchar(255)" remarks="密码">
               <constraints  nullable="false"/>
             </column>
           </createTable>
           <modifySql dbms="mysql">
             <append value="ENGINE=INNODB DEFAULT CHARSET utf8mb4 COLLATE utf8mb4_general_ci"/>
           </modifySql>
         </changeSet>
     ```          
2. 创建索引
     ```
        <createIndex tableName="ven_vendor_info" indexName="idx_vendor_id">
           <column name="vendor_id" />
        </createIndex>  
     ```
3. 新增列
     ```
       <addColumn tableName="users">
         <column name="name" type="varchar(255)" remarks="名称">
           <constraints  nullable="false"/>
         </column>
       </addColumn>
     ```
4. 删除列
     ```
      <dropColumn tableName="atl_relation_vendor_company" columnName="is_deleted"/>
     ```
5. 修改数据类型
     ```
       <modifyDataType tableName="atl_vendor_info" columnName="import_code" newDataType="varchar(50)" />
     ```
6. 重命名列
     ```
       <renameColumn tableName="atl_vendor_type" oldColumnName="is_deleted" newColumnName="deleted"/>
     ```
## 集成SpringBoot
1. 引入依赖
   ```
       <dependencies>
           <dependency>
               <groupId>org.liquibase</groupId>
               <artifactId>liquibase-core</artifactId>
           </dependency>
           <dependency>
       </dependencies>
   ```
2. yml文件中配置
   ```
     spring:
       datasource:
         url: jdbc:mysql://127.0.0.1:3306/snow?characterEncoding=utf8&useSSL=false
         username: root
         password: 123456
         driverClassName: com.mysql.jdbc.Driver
         type: com.alibaba.druid.pool.DruidDataSource
       liquibase:
             enabled: true
             change-log: "classpath:liquibase/change_log/index.xml"
             contexts: dev
   ```
3. 变更集,即对数据库的操作
   ```
     <databaseChangeLog
             xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
             http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.4.xsd">
       <property name="autoIncrement" value="true" dbms="mysql"/>
       <changeSet id="init-schema" author="17835059864@163.com" >
         <comment>init schema</comment>
         <createTable tableName="users" remarks="用户表">
           <column name="id" type="bigint" autoIncrement="${autoIncrement}" remarks="主键">
             <constraints primaryKey="true" nullable="false"/>
           </column>
           <column name="name" type="varchar(255)" remarks="名称">
             <constraints  nullable="false"/>
           </column>
           <column name="password" type="varchar(255)" remarks="密码">
             <constraints  nullable="false"/>
           </column>
         </createTable>
         <modifySql dbms="mysql">
           <append value="ENGINE=INNODB DEFAULT CHARSET utf8mb4 COLLATE utf8mb4_general_ci"/>
         </modifySql>
       </changeSet>
       <changeSet author="diff-generated" id="1185214997195-7">
           <createIndex indexName="PK_EMP" tableName="EMP">
               <column name="EMPNO"/>
           </createIndex>
       </changeSet>
     </databaseChangeLog>
   ```
4. index.xml,封装一个共用的入口，通过include 引入变更集
   ```
        <?xml version="1.0" encoding="UTF-8"?>
        <databaseChangeLog
                xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
                xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
                 http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.1.xsd">
          <include file="classpath:liquibase/change_log/2019-10-29-init-schema.xml" relativeToChangelogFile="false"/>
        </databaseChangeLog>
   ```
5. 注入Bean,放入启动类中，或者新建配置文件，添加@Configuration。
   ```
         @Bean
         public SpringLiquibase springLiquibase(DataSource dataSource) {
             SpringLiquibase springLiquibase = new SpringLiquibase();
             springLiquibase.setDataSource(dataSource);
             springLiquibase.setChangeLog("classpath:/liquibase/index.xml");
             return springLiquibase;
         }
   ```
## 参考地址
  [Liquibase](http://www.liquibase.org/documentation/generating_changelogs.html)
  ## demo地址
  [demo地址](https://github.com/florarose/com.merry.tool/tree/master/SpringBoot-Liquibase)
  欢迎关注公众号，查看更多内容 ： 
  ![XG54_9_WXMH_5X_HB_H_7V](https://yqfile.alicdn.com/17479bd1026b3d93f5718893256adf7d6d164e5d.png)