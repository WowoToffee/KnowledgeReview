# maven 第三方插件打包

## 普通java项目打包（无配置）

POM.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>powder-tcp-forwarding</artifactId>
    <version>1.0-SNAPSHOT</version>

    ....

    <build>
        <plugins>
            <plugin>
                 <!-- 指定maven编译的jdk版本,如果不指定,maven3默认用jdk 1.5 maven2默认用jdk1.3 -->
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <!-- 源代码使用的JDK版本 -->   
                    <source>1.8</source>
                    <!-- 需要生成的目标class文件的编译版本 --> 
                    <target>1.8</target>
                    <!-- 字符集编码 -->
                    <encoding>UTF-8</encoding>
                    <!-- 这个选项用来传递编译器自身不包含但是却支持的参数选项 -->  
                    <compilerArguments>
                        <extdirs>${project.basedir}/lib</extdirs>
                    </compilerArguments>
                </configuration>
            </plugin>
            <plugin>
                <!--shade插件绑定在maven的package阶段，他会将项目依赖的jar包解压并融合到项目自身编译文件中。-->
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>2.3</version>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                        <configuration>
                            <transformers>
                                <transformer
                                        implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                     <!--启动类-->
                                    <mainClass>com.nengfengdigital.powder.server.CollectionNettyServer</mainClass>
                                </transformer>
                            </transformers>
                        </configuration>
                    </execution>
                </executions>
            </plugin>

        </plugins>
        <defaultGoal>compile</defaultGoal>
    </build>


</project>

```

## spring boot 打zip 包



## spring boot 打docker包

```xml
<build>
    <finalName>${project.artifactId}</finalName>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
        <plugin>
            <groupId>io.fabric8</groupId>
            <artifactId>docker-maven-plugin</artifactId>
            <version>0.26.1</version>
            <configuration>
                <dockerHost>unix:///var/run/docker.sock</dockerHost>
                <images>
                    <image>
                        <name>snexus.stylefeng.cn:6001/${project.artifactId}:${docker.img.version}</name>
                        <build>
                            <from>java:8</from>
                            <assembly>
                                <descriptor>docker-assembly.xml</descriptor>
                            </assembly>
                            <cmd>
                                <shell>java -agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=50551 -jar -Xms128m -Xmx128m -Xss1024K -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=128m /maven/${project.artifactId}.jar</shell>
                            </cmd>
                        </build>
                    </image>
                </images>
            </configuration>
        </plugin>
    </plugins>
</build>
```

`docker-assembly.xml`

```xml
<assembly xmlns="http://maven.apache.org/plugins/maven-assembly-plugin/assembly/1.1.2"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/plugins/maven-assembly-plugin/assembly/1.1.2 http://maven.apache.org/xsd/assembly-1.1.2.xsd">
    <files>
        <file>
            <source>target/${project.artifactId}.jar</source>
            <destName>${project.artifactId}.jar</destName>
        </file>
    </files>
</assembly>
```

