## IDEA 远程debug Docker 容器内 tomcat 服务流程

### 远程服务部署

#### 环境依赖

- docker image : tomcat:9.0.22-jdk11-openjdk-slim
- war包 ： 这里是 demo.war
以下是具体的DockerFile编写，可以依据自己的proj适当修改
```dockerfile
FROM tomcat:9.0.22-jdk11-openjdk-slim

# war package
ENV BASE="/usr/local" RUN_HOME="tomcat" PROG_DIR="webapps" MODULE="demo module" 

ENV CUSTOME_CLASSPATH /customeClassPath

ENV CLASSPATH $CUSTOME_CLASSPATH:$CLASSPATH

# 特别需要注意address配置，这里表示1043端口是开放给外部使用的远程debug 端口
ENV JPDA_OPTS="-agentlib:jdwp=transport=dt_socket,address=*:1043,server=y,suspend=n"
# 将当前目录的war包映射到容器中相应的目录中去
ADD ./demo.war ${BASE}/${RUN_HOME}/${PROG_DIR}
```

#### 执行流程

- 在服务器的任意目录下准备好上面的 Dockerfile 以及 相应的 war 包（demo.war）

- 执行如下命令生成Docker image

  ```dockerfile
  docker build -t demo:v1       # docker build -t [imageNmae:version]   
  ```
  
- 根据生成的Docker image 来生成有效的container，注意这里是tomcat web服务端口8080 映射成了5066，将上面JPDA的debug端口1043映射成了5065

  ```dockerfile
  docker run -dit -p 5066:8080 -p 5065:1043 --name [newContainerName] [imageID]
  ```

- 进入到容器中

  ```dockerfile
  docker exec -it [containerID] bash
  ```

- 进入到tomcat的bin目录下， 执行catalina.sh jada start 来以调试模式运行jvm

  ```bash
  cd $tomcat/bin
  ./catalina.sh jada start
  ```

至此你的服务器端的服务已经部署完毕。

以上的流程都可以通过docker-compose 来一站式实现，这里就留给大家做尝试。



### 本地IDEA调试

- 获取部署的服务的代码，务必保持代码一致。

- 点击edit configuration， 配置远程服务器的id, port，注意这里的port配置的映射出来的debug端口

  ![image-20190728145521541](/Users/caiguoqing/Library/Application Support/typora-user-images/image-20190728145521541.png)

- 设置断点，启动 debug 模式，通过访问相应的web服务就可以实现远程debug了。



