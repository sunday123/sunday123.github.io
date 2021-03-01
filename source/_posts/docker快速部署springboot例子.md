---
title: docker快速部署springboot例子
date: 2020-10-14 18:55:23
tags: ['docker','springboot']
---
## docker快速部署springboot项目例子

前提:linux安装了docker和maven

### 步骤一：docker安装jdk

查找镜像

```shell
docker search  jdk 
```

从查找的里面拉取一个

```shell
docker  pull jdk镜像名
```

查看

```shell
docker run --rm gmaslowski/jdk:lastest java -version
```

![](/../../../../images/2020-10/dokcer16026630951830.png)



### 步骤二：配置pom和写Dockerfile

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.4.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.example</groupId>
    <artifactId>demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>demo</name>
    <description>Demo project for Spring Boot</description>

    <properties>
    
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <docker.image.prefix>springdemo</docker.image.prefix>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        
        <!-- https://mvnrepository.com/artifact/com.spotify/docker-client -->
<dependency>
<groupId>com.spotify</groupId>
<artifactId>docker-maven-plugin</artifactId>
<version>1.0.0</version>
</dependency>

    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>


            <plugin>
                <groupId>com.spotify</groupId>
                <artifactId>docker-maven-plugin</artifactId>
                <version>1.0.0</version>
                <configuration>
                    <imageName>${docker.image.prefix}/${project.artifactId}</imageName>
                    <dockerDirectory>src/main/docker</dockerDirectory>
                    <resources>
                        <resource>
                            <targetPath>/</targetPath>
                            <directory>${project.build.directory}</directory>
                            <include>${project.build.finalName}.jar</include>
                        </resource>
                    </resources>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>

```

开放docker端口2375

```shell
vim /usr/lib/systemd/system/docker.service
```

修改

```shell
ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix://var/run/docker.sock \
```

重新加载docker配置

```shell
systemctl daemon-reload //加载docker守护线程
systemctl restart docker //重启docker
```

Dockerfile

390b58b1be42是docker里面已经部署好的jdk镜像ID

```
FROM 390b58b1be42
VOLUME /tmp
ADD demo-0.0.1-SNAPSHOT.jar app.jar
RUN sh -c 'touch /app.jar'
EXPOSE 8002
ENV JAVA_OPTS=""
ENTRYPOINT [ "sh", "-c", "java $JAVA_OPTS -Djava.security.egd=file:/dev/./urandom -jar /app.jar" ]
```

把项目传到linux系统里/home目录下



### 步骤三：生成项目镜像

项目根目录执行

```shell
mvn package docker:build
```

![](/../../../../images/2020-10/docker16026630484958.png)

![](/../../../../images/2020-10/docker16026630484959.png)

查看，如第一个demo是生成的镜像

![](/../../../../images/2020-10/docker16026629178073.png)



### 步骤四：启动

```shell
docker run -p 8002:8002 -t springdemo/demo
```

![](/../../../../images/2020-10/docker16026629504716.png)

部署成功打开查看http://127.0.0.1:8002



[本例子->下载](https://github.com/sunday123/DockerDemo)