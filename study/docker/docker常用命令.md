# Docker常用笔记

## 从docker运行jar包

新建DockerFile文件，填入如下内容。 替换japp.jar为实际的jar文件名

```dockerfile
FROM eclipse-temurin:11
RUN mkdir /opt/app
COPY japp.jar /opt/app
CMD ["java", "-jar", "/opt/app/japp.jar"]
//加更多jar运行参数 
//CMD ["java", "-jar", "/opt/app/japp.jar","192.168.196.69"]
```

构建、运行

```console
docker build -t japp .
docker run -it --rm japp

//后台运行
//docker run -d --rm japp
```

