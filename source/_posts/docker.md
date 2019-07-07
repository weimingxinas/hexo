---
title: docker
date: 2019-07-06 14:24:10
tags: docker
---
最近遇到的项目都是需要手动打包部署的，很烦，学一学doker和jenkins配合nginx来做自动构建和部署  
1. 使用yum安装   yum -y install docker-io
2. **docker version**查看版本
3. **docker image pull hello-world** 将image文件拉下来
4. **docker container run hello-world**运行该image
5.  
> - 列出本机正在运行的容器  
> $ **docker container ls**  
>-  列出本机所有容器，包括终止运行的容器  
>$ **docker container ls --all**
6. **docker container kill [containID]** 终止该容器
7. **docker container rm [containerID]** 删除容器文件 
8. image 文件生成的容器实例，本身也是一个文件，称为容器文件。也就是说，一旦容器生成，就会同时存在两个文件： image文件和容器文件。而且关闭容器并不会删除容器文件，只是容器停止运行而已。  
##### 编写dockerfile文件
1. 首先，在项目的根目录下，新建一个文本文件.dockerignore，写入下面的内容  
```
.git
node_modules
npm-debug.log
```
2. 在项目的根目录下，新建一个文本文件 Dockerfile，写入下面的内容  

```
FROM node:8.4
COPY . /app
WORKDIR /app
RUN npm install --registry=https://registry.npm.taobao.org
EXPOSE 3000
```
- FROM node:8.4：该 image 文件继承官方的 node image，冒号表示标签，这里标签是8.4，即8.4版本的 node。

- COPY . /app：将当前目录下的所有文件（除了.dockerignore排除的路径），都拷贝进入 image 文件的/app目录。

- WORKDIR /app：指定接下来的工作路径为/app。

- RUN npm install：在/app目录下，运行npm install命令安装依赖。注意，安装后所有的依赖，都将打包进入 image 文件。

- EXPOSE 3000：将容器 3000 端口暴露出来， 允许外部连接这个端口。
##### 有用的命令
1. **docker container start**   
前面的docker container run命令是新建容器，每运行一次，就会新建一个容器。同样的命令运行两次，就会生成两个一模一样的容器文件。如果希望重复使用容器，就要使用docker container start命令，它用来**启动已经生成、已经停止运行的容器文件。**
2. **docker container stop**   
前面的docker container kill命令终止容器运行，相当于向容器里面的主进程发出 SIGKILL 信号。而docker container stop命令也是用来终止容器运行，相当于向容器里面的主进程发出 SIGTERM 信号，然后过一段时间再发出 SIGKILL 信号。  
这两个信号的差别是，应用程序收到 SIGTERM 信号以后，可以自行进行收尾清理工作，但也可以不理会这个信号。如果收到 SIGKILL 信号，就会强行立即终止，那些正在进行中的操作会全部丢失。
3. **docker container exec**  
``

docker container exec命令用于进入一个正在运行的 docker 容器。如果docker run命令运行容器的时候，没有使用-it参数，就要用这个命令进入容器。一旦进入了容器，就可以在容器的 Shell 执行命令了。 
4. **docker container cp**  
docker container cp命令用于从正在运行的 Docker 容器里面，将文件拷贝到本机。下面是拷贝到当前目录的写法。

>$ docker container cp [containID]:[/path/to/file]  .     

##### run和cmd的区别
RUN命令在 image 文件的构建阶段执行，执行结果都会打包进入 image 文件；CMD命令则是在容器启动后执行。另外，一个 Dockerfile 可以包含多个RUN命令，但是只能有一个CMD命令
##### docker安装jenkins  
百度。。。   
1. **docker ps**  
查看运行的docker
2. **docker ps -a**  
查看所有docker，包含挂掉的  
3. **docker logs [id]**  
查看挂掉的原因（日志）  
##### jenkins和github

```
docker container run -itd -p 8903:8080 --name jenkins01 --privileged=true  -v /home/wmxDocker/jenkins:/var/jenkins_home jenkins

```


