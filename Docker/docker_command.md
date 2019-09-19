## 一、基本命令
    
   - docker pull	获取image
   - docker build	创建image
   - docker images	列出image
   - docker run	运行container
   - docker ps	列出container
   - docker stop 停止container
   - docker rm	删除container
   - docker rmi	删除image
   - docker cp	在主机和容器之间之间拷贝文件
   - docker commit	保存改动成为新的image
## 二、 docker run (Volume操作) 容器和宿主机目录挂载的三种方式：
  ### 1 docker run -d --name Mynginx -v /usr/share/nginx/html nginx_snow
   - -d : 后台运行
   - --name：指定名称
   - -v /usr/share/nginx/html：运行容器内部用来访问网页的地址
   - nginx_snow: 容器名称
  ### 2 docker run -p 8080:80 --name Mynginx -v $PWD:/usr/share/nginx/html -d nginx_snow
   - $PWD:/usr/share/nginx/html: 将当前目录挂载到容器/usr/share/nginx/html目录
  ### 3 
   
   #### 使用 docker create 创建一个新的容器但不启动它： docker create -v $PWD/Docker:/var/docker --name docker_container ubuntu
      
   - $PWD/Docker : 宿主机目录
   - /var/docker : docker目录
   - docker_container : 容器名
   -  ubuntu : 基础镜像
      
   #### 启动 ubuntu 容器镜像（默认ubuntu基础镜像没有服务）：docker run -it --volumes-from docker_container ubuntu /bin/bash
   -   -it: 以交互模式运行容器，并为容器重新分配一个伪输入终端
   -   --volumes-from data_container：以另外一个容器挂载
   -   最后在容器内执行/bin/bash命令
## 三、 Dockfile 命令
   FROM	base image
   RUN	执行命令
   ADD	添加文件
   COPY	拷贝文件
   CMD	执行文件
   EXPOSE	暴露端口
   WORKDIR	指定路径
   MAINTAINER	维护者
   ENV	设定环境
   ENTRYPOINT	容器入口
   USER	指定用户
   VOLUME	mount point
   docker inspect 容器id或者名称 查看容器的元数据
   mount 执行docker exec -it 容器id /bin/bash 后，执行mount，可以查看容器的挂载