## 一、 执行命令，删除原有的docker
        ```
            sudo yum remove docker \
                              docker-client \
                              docker-client-latest \
                              docker-common \
                              docker-latest \
                              docker-latest-logrotate \
                              docker-logrotate \
                              docker-selinux \
                              docker-engine-selinux \
                              docker-engine
         ```
##  二、 安裝docker
   - sudo yum install docker-ce 安装
   - sudo systemctl start docker 启动
   - docker --version  查看版本
    
   ### 1 安装maven,和java
           
   - docker pull hub.c.163.com/wuxukun/maven-aliyun:3-jdk-8 
   
   ----
   ### 2 安装mysql
   - 拉取镜像，docker pull mysql:5.8
   - 安装mysql主从复制
   #### 安装主库 
   - docker run --name mysql-master -e MYSQL_ROOT_PASSWORD=root -p 10000:3306 -d mysql 
   - docker ps 查看mysql容器是否启动
   ##### 配置
   - 使用navicat 连接数据库，报错。Client does not support authentication protocol ； 解决方法 如下：
  

   
   1. 授权
        GRANT ALL ON *.* TO 'root'@'%';
   2. 刷新权限
        flush privileges;
    　此时,还不能远程访问,因为Navicat只支持旧版本的加密,需要更改mysql的加密规则;
   
   3. 更改加密规则
       ALTER USER 'root'@'localhost' IDENTIFIED BY 'password' PASSWORD EXPIRE NEVER;
   4. 更新root用户密码
       ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '123456';
   5. 刷新权限
       flush privileges;
   - docker exec -it mysql-master /bin/bash 或 docker exec -it 627a2368c865 /bin/bash 进入mysql主库容器内部
   - 切换到/etc/mysql 目录下， cd /etc/mysql .
   - 修改my.cnf文件，vim my.cnf 在文件中添加如下内容
   
     ```  
      [mysqld]
      ## 同一局域网内注意要唯一
      server-id=1001  
      ## 开启二进制日志功能，可以随便取（关键）
      log-bin=mysql-bin
     ```  
       >>如果报错，出现没有vi 命令： 依次执行以下命令
       
        ```  
          apt-get update;
          apt-get install vim
        ```  
   - 重启mysql服务，使用命令： docker restart  容器名称或者id (1b4671904bfa)
   
   ####  创建数据同步用户，授予用户 slave REPLICATION SLAVE权限和REPLICATION CLIENT权限，用于在主从库之间同步数据。
   
   - CREATE USER'slave'@'%'IDENTIFIED BY'123456';
   - GRANT REPLICATION SLAVE,REPLICATION CLIENT ON*.*TO'slave'@'%';
   #### 安装从库 
   - docker run --name mysql-slave -e MYSQL_ROOT_PASSWORD=123456 -p 10001:3306 -d mysql
   - docker ps 查看mysql容器是否启动
   ##### 配置    
  - docker exec -it mysql-slave /bin/bash 或 docker exec -it 627a2368c865 /bin/bash 进入mysql主库容器内部
  - 切换到/etc/mysql 目录下， cd /etc/mysql .
  - 修改my.cnf文件，vim my.cnf 在文件中添加如下内容
  
    ```  
     [mysqld]
        server-id=101  
        ## 开启二进制日志功能
        log-bin=mysql-slave-bin   
        ## relay_log配置中继日志
        relay_log=edu-mysql-relay-bin
    ```  
      >>如果报错，出现没有vi 命令： 依次执行以下命令
      
       ```  
         apt-get update;
         apt-get install vim
       ```  
  - 重启mysql服务，使用命令： docker restart  容器名称或者id (1b4671904bfa)
  ### 配置主从复制
  - 登录主服务器，执行show master status;如下图
  - 登录从服务器，执行 
     ```  
       change master to master_host='172.17.0.2',master_user='slave',
       master_password='123456',master_log_file='mysql-bin.000001',
       master_log_pos=155,master_port=3306;
     ```  
  - show slave status \G;用于查看主从同步状态。
  - start slave ; 启动主从复制。
  - show slave status \G;用于查看主从同步状态。
  - 查看容器的ip docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' 容器名称或id(mysql-master)
  - 在容器中安装ping : apt-get install iputils-ping
  - 参数解释
     1. master_host ：Master的地址，指的是容器的独立ip,
     2. master_port：Master的端口号，指的是容器的端口号
     3. master_user：用于数据同步的用户
     4. master_password：用于同步的用户的密码
     5. masterlogfile：指定 Slave 从哪个日志文件开始复制数据
     6. masterlogpos：从哪个 Position 开始读，
     7. masterconnectretry：如果连接失败，重试的时间间隔，单位是秒，默认是60秒
  ### 3 安装 redis
  
  - 使用Dockfile 安装redis
      ```
     FROM centos
     MAINTAINER merry
     WORKDIR /home
     RUN yum install -y wget gcc && \      
         rpm --rebuilddb && \
         yum -y install gcc automake autoconf libtool make && \
         yum -y install net-tools && \
         yum -y install tar && \
         wget http://download.redis.io/redis-stable.tar.gz && \
         tar -xvzf redis-stable.tar.gz && \
         mv redis-stable/ redis && \
         rm -f redis-stable.tar.gz && \
         yum clean all && \
         cd redis && \
         make && make install     
     EXPOSE 6379
     ENTRYPOINT redis-server /home/redis/redis.conf
     CMD ["redis-server"]
    
     ```  
  - 构建: docker build -t redis:v1 .
  - 启动： docker run -d -p 6378:6379 redis:v1
  ### 4 安装 nginx
  - 使用Dockfile 安装
  
    ```  
        FROM centos
        MAINTAINER merry
        
        #ENV 设置环境变量
        ENV PATH /usr/local/nginx/sbin:$PATH
        
        ADD http://nginx.org/download/nginx-1.16.0.tar.gz .
         
        #RUN 执行以下命令 
        RUN yum install -y pcre-devel wget net-tools gcc zlib zlib-devel make openssl-devel
        RUN useradd -M -s /sbin/nologin nginx
        RUN tar -zxvf nginx-1.16.0.tar.gz
        RUN mkdir -p /usr/local/nginx
        RUN cd nginx-1.16.0 && ./configure --prefix=/usr/local/nginx --user=nginx --group=nginx --with-http_stub_status_module && make && make install
        ADD nginx.conf /usr/local/nginx/conf/nginx.conf
         
        #EXPOSE 映射端口
        EXPOSE 80 
        EXPOSE 443
        #ENTRYPOINT 运行以下命令
        ENTRYPOINT ["nginx"]
 
    ```  
  - 构建： docker build --rm --tag centos_nginx:centos .
  - 启动容器： 
    ```  
          docker run \
          --name centos_nginx \
          -d -p 80:80 \
          -v  /home/nginx/html:/usr/share/nginx/html \
          -v  /home/log:/var/log/nginx \
          -v  /home/nginx.conf:/usr/local/nginx/nginx.conf:ro \
          -v  /home/nginx/conf.d:/usr/local/nginx/conf.d \
          nginx 
          
    ```  

   
         
         
         
