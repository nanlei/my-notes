<h3 align="center"><b>02 - 固化的Maven依赖</b></h3>

之前已经使用的```org.springframework.boot:spring-boot-starter-web```和```org.springframework.boot:spring-boot-loader```均继承自```org.springframework.boot:spring-boot-starter-parent```，比如版本信息，这些是Maven依赖管理的范畴，降低了Spring Boot依赖管理的成本。

而配置```org.springframework.boot:spring-boot-starter-parent```也存在一定的限制，因为是单继承方式，限制了其依赖仅是Spring Boot相关。若要需要固化其他类型的依赖就比较麻烦，或者应用的```pom.xml```有自定义的parent。

在官方文档 (https://docs.spring.io/spring-boot/docs/current/maven-plugin/reference/html/) 中，介绍了相关内容：

><b>Using Spring Boot without the Parent POM</b>  
There may be reasons for you not to inherit from the ```spring-boot-starter-parent``` POM. You may have your own corporate standard parent that you need to use or you may prefer to explicitly declare all your Maven configuration.  
>
>If you do not want to use the ```spring-boot-starter-parent```, you can still keep the benefit of the dependency management (but not the plugin management) by using an ```import``` scoped dependency, as follows:
>```xml 
><dependencyManagement>
>	<dependencies>
>		<dependency>
>			<!-- Import dependency management from Spring Boot -->
>			<groupId>org.springframework.boot</groupId>
>			<artifactId>spring-boot-dependencies</artifactId>
>			<version>2.3.2.RELEASE</version>
>			<type>pom</type>
>			<scope>import</scope>
>		</dependency>
>	</dependencies>
></dependencyManagement>
>```

对示例项目作出调整，修改```pom.xml```文件：
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>deep-in-spring-boot</groupId>
    <artifactId>first-spring-boot-application</artifactId>
    <packaging>war</packaging>
    <version>1.0.0-SNAPSHOT</version>
    <name>first-spring-boot-application</name>

<!--    <parent>-->
<!--        <groupId>org.springframework.boot</groupId>-->
<!--        <artifactId>spring-boot-starter-parent</artifactId>-->
<!--        <version>2.3.2.RELEASE</version>-->
<!--    </parent>-->


    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <version>2.3.2.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>3.8.1</version>
            <scope>test</scope>
        </dependency>
        <!-- spring-boot-loader 分析 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-loader</artifactId>
            <scope>provided</scope>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <!-- Import dependency management from Spring Boot -->
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>2.3.2.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

去掉```<parent>```元素，添加了```spring-boot-dependencies```，之后重新打包：
```cmd
$ mvn clean package
[INFO] Scanning for projects...
[INFO] 
[INFO] ---------< deep-in-spring-boot:first-spring-boot-application >----------
[INFO] Building first-spring-boot-application 1.0.0-SNAPSHOT
[INFO] --------------------------------[ war ]---------------------------------
[INFO] 
[INFO] --- maven-clean-plugin:2.5:clean (default-clean) @ first-spring-boot-application ---
[INFO] Deleting /Users/nanlei/Dev/Workspace/first-spring-boot-application/target
[INFO] 
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ first-spring-boot-application ---
[WARNING] Using platform encoding (UTF-8 actually) to copy filtered resources, i.e. build is platform dependent!
[INFO] skip non existing resourceDirectory /Users/nanlei/Dev/Workspace/first-spring-boot-application/src/main/resources
[INFO] 
[INFO] --- maven-compiler-plugin:3.1:compile (default-compile) @ first-spring-boot-application ---
[INFO] Changes detected - recompiling the module!
[WARNING] File encoding has not been set, using platform encoding UTF-8, i.e. build is platform dependent!
[INFO] Compiling 1 source file to /Users/nanlei/Dev/Workspace/first-spring-boot-application/target/classes
[INFO] 
[INFO] --- maven-resources-plugin:2.6:testResources (default-testResources) @ first-spring-boot-application ---
[WARNING] Using platform encoding (UTF-8 actually) to copy filtered resources, i.e. build is platform dependent!
[INFO] skip non existing resourceDirectory /Users/nanlei/Dev/Workspace/first-spring-boot-application/src/test/resources
[INFO] 
[INFO] --- maven-compiler-plugin:3.1:testCompile (default-testCompile) @ first-spring-boot-application ---
[INFO] Changes detected - recompiling the module!
[WARNING] File encoding has not been set, using platform encoding UTF-8, i.e. build is platform dependent!
[INFO] Compiling 1 source file to /Users/nanlei/Dev/Workspace/first-spring-boot-application/target/test-classes
[INFO] 
[INFO] --- maven-surefire-plugin:2.12.4:test (default-test) @ first-spring-boot-application ---
[INFO] Surefire report directory: /Users/nanlei/Dev/Workspace/first-spring-boot-application/target/surefire-reports

-------------------------------------------------------
 T E S T S
-------------------------------------------------------
Running deep.in.spring.boot.AppTest
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.004 sec

Results :

Tests run: 1, Failures: 0, Errors: 0, Skipped: 0

[INFO] 
[INFO] --- maven-war-plugin:2.2:war (default-war) @ first-spring-boot-application ---
[INFO] Packaging webapp
[INFO] Assembling webapp [first-spring-boot-application] in [/Users/nanlei/Dev/Workspace/first-spring-boot-application/target/first-spring-boot-application-1.0.0-SNAPSHOT]
[INFO] Processing war project
[INFO] Webapp assembled in [133 msecs]
[INFO] Building war: /Users/nanlei/Dev/Workspace/first-spring-boot-application/target/first-spring-boot-application-1.0.0-SNAPSHOT.war
[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  1.686 s
[INFO] Finished at: 2020-08-04T11:11:26+08:00
[INFO] ------------------------------------------------------------------------
[ERROR] Failed to execute goal org.apache.maven.plugins:maven-war-plugin:2.2:war (default-war) on project first-spring-boot-application: Error assembling WAR: webxml attribute is required (or pre-existing WEB-INF/web.xml if executing in update mode) -> [Help 1]
[ERROR] 
[ERROR] To see the full stack trace of the errors, re-run Maven with the -e switch.
[ERROR] Re-run Maven using the -X switch to enable full debug logging.
[ERROR] 
[ERROR] For more information about the errors and possible solutions, please read the following articles:
[ERROR] [Help 1] http://cwiki.apache.org/confluence/display/MAVEN/MojoExecutionException
```

打包失败了，错误原因是web应用的部署描述文件```WEB-INF/web.xml```在项目中并不存在，当设置```pom.xml```的```<packaging>war</packaging>```时，Maven会执行```maven-war-plugin:2.2```，```WEB-INF/web.xml```就是必须的。但是，在作出调整之前却没有这个问题，那么问题就在```spring-boot-dependencies```和```spring-boot-starter-parent```的差别之中。

查看```spring-boot-starter-parent-2.3.2.RELEASE.pom```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <modelVersion>4.0.0</modelVersion>
  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-dependencies</artifactId>
    <version>2.3.2.RELEASE</version>
  </parent>
  <artifactId>spring-boot-starter-parent</artifactId>
  <packaging>pom</packaging>
  <name>spring-boot-starter-parent</name>
  <description>Parent pom providing dependency and plugin management for applications built with Maven</description>
  ...
</project>
```

可以看出```spring-boot-dependencies```是```spring-boot-starter-parent```的```parent```，也就是说可以将```spring-boot-dependencies```直接作为```parent```。

查看```spring-boot-dependencies-2.3.2.RELEASE.pom```，可以看到
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <modelVersion>4.0.0</modelVersion>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-dependencies</artifactId>
  <version>2.3.2.RELEASE</version>
  <packaging>pom</packaging>
  <name>spring-boot-dependencies</name>
  <description>Spring Boot Dependencies</description>
  ...
  <properties>
    ...
    <spring-boot.version>2.3.2.RELEASE</spring-boot.version>
    ...
    <maven-war-plugin.version>3.2.3</maven-war-plugin.version>
    ...
  </properties>
  <dependencyManagement>
      ...
      <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <version>${spring-boot.version}</version>
      </dependency>
      ...
  </dependencyManagement>
  <build>
    <pluginManagement>
      <plugins>
        ...
        <plugin>
          <groupId>org.apache.maven.plugins</groupId>
          <artifactId>maven-war-plugin</artifactId>
          <version>${maven-war-plugin.version}</version>
        </plugin>
        ...
      </plugins>
    </pluginManagement>
  </build>
</project>
```

无论是Spring Boot的依赖```spring-boot-starter-web```等，还是插件```maven-war-plugin```等，均定义在此处。当项目```pom.xml```文件用```<parent>```方式引入```spring-boot-starter-parent```时，```mvn package```将使用```maven-war-plugin:3.2.3```和```spring-boot-maven-plugin:2.3.2.RELEASE```。

相反，```<dependencyManagement>```方式导入```spring-boot-dependencies```尽管和前者相同，但此时仅关注```<dependencyManagement>```，所以```maven-war-plugin```采用的版本是2.2，手动添加```maven-war-plugin:3.2.3```到项目```pom.xml```中：
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    ...
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <version>2.3.2.RELEASE</version>
            </plugin>
            <!-- 保持和spring-boot-dependencies版本一致 -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-war-plugin</artifactId>
                <version>3.2.3</version>
            </plugin>
        </plugins>
    </build>
</project>
```

再次执行打包：
```cmd
$ mvn clean package
[INFO] Scanning for projects...
[INFO] 
[INFO] ---------< deep-in-spring-boot:first-spring-boot-application >----------
[INFO] Building first-spring-boot-application 1.0.0-SNAPSHOT
[INFO] --------------------------------[ war ]---------------------------------
[INFO] 
[INFO] --- maven-clean-plugin:2.5:clean (default-clean) @ first-spring-boot-application ---
[INFO] Deleting /Users/nanlei/Dev/Workspace/first-spring-boot-application/target
[INFO] 
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ first-spring-boot-application ---
[WARNING] Using platform encoding (UTF-8 actually) to copy filtered resources, i.e. build is platform dependent!
[INFO] skip non existing resourceDirectory /Users/nanlei/Dev/Workspace/first-spring-boot-application/src/main/resources
[INFO] 
[INFO] --- maven-compiler-plugin:3.1:compile (default-compile) @ first-spring-boot-application ---
[INFO] Changes detected - recompiling the module!
[WARNING] File encoding has not been set, using platform encoding UTF-8, i.e. build is platform dependent!
[INFO] Compiling 1 source file to /Users/nanlei/Dev/Workspace/first-spring-boot-application/target/classes
[INFO] 
[INFO] --- maven-resources-plugin:2.6:testResources (default-testResources) @ first-spring-boot-application ---
[WARNING] Using platform encoding (UTF-8 actually) to copy filtered resources, i.e. build is platform dependent!
[INFO] skip non existing resourceDirectory /Users/nanlei/Dev/Workspace/first-spring-boot-application/src/test/resources
[INFO] 
[INFO] --- maven-compiler-plugin:3.1:testCompile (default-testCompile) @ first-spring-boot-application ---
[INFO] Changes detected - recompiling the module!
[WARNING] File encoding has not been set, using platform encoding UTF-8, i.e. build is platform dependent!
[INFO] Compiling 1 source file to /Users/nanlei/Dev/Workspace/first-spring-boot-application/target/test-classes
[INFO] 
[INFO] --- maven-surefire-plugin:2.12.4:test (default-test) @ first-spring-boot-application ---
[INFO] Surefire report directory: /Users/nanlei/Dev/Workspace/first-spring-boot-application/target/surefire-reports

-------------------------------------------------------
 T E S T S
-------------------------------------------------------
Running deep.in.spring.boot.AppTest
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.008 sec

Results :

Tests run: 1, Failures: 0, Errors: 0, Skipped: 0

[INFO] 
[INFO] --- maven-war-plugin:3.2.3:war (default-war) @ first-spring-boot-application ---
[INFO] Packaging webapp
[INFO] Assembling webapp [first-spring-boot-application] in [/Users/nanlei/Dev/Workspace/first-spring-boot-application/target/first-spring-boot-application-1.0.0-SNAPSHOT]
[INFO] Processing war project
[INFO] Webapp assembled in [110 msecs]
[INFO] Building war: /Users/nanlei/Dev/Workspace/first-spring-boot-application/target/first-spring-boot-application-1.0.0-SNAPSHOT.war
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  2.441 s
[INFO] Finished at: 2020-08-04T13:40:35+08:00
[INFO] ------------------------------------------------------------------------
```

此时打包可以成功，启动程序：
```cmd
$ java -jar target/first-spring-boot-application-1.0.0-SNAPSHOT.war 
no main manifest attribute, in target/first-spring-boot-application-1.0.0-SNAPSHOT.war
```

没有发现```MANIFEST.MF```，说明```spring-boot-maven-plugin```没有执行，官方文档对此有详细说明：
><b>Create an Executable JAR with Maven</b>  
The ```spring-boot-maven-plugin``` can be used to create an executable “fat” JAR. If you use the ```spring-boot-starter-parent``` POM, you can declare the plugin and your jars are repackaged as follows:
>```xml
><build>
>    <plugins>
>        <plugin>
>            <groupId>org.springframework.boot</groupId>
>            <artifactId>spring-boot-maven-plugin</artifactId>
>        </plugin>
>    </plugins>
></build>
>```
>
>If you do not use the parent POM, you can still use the plugin. However, you must additionally add an ```<executions>``` section, as follows:
>```xml
><build>
>   <plugins>
>        <plugin>
>            <groupId>org.springframework.boot</groupId>
>            <artifactId>spring-boot-maven-plugin</artifactId>
>            <version>2.3.2.RELEASE</version>
>            <executions>
>                <execution>
>                    <goals>
>                        <goal>repackage</goal>
>                    </goals>
>                </execution>
>            </executions>
>        </plugin>
>    </plugins>
></build>
>```

对```pring-boot-maven-plugin```设置```execution```的```goal```为```repackage```后，再次打包：
```cmd
$ mvn clean package
[INFO] Scanning for projects...
(省略部分内容...)
[INFO] --- maven-war-plugin:3.2.3:war (default-war) @ first-spring-boot-application ---
[INFO] Packaging webapp
[INFO] Assembling webapp [first-spring-boot-application] in [/Users/nanlei/Dev/Workspace/first-spring-boot-application/target/first-spring-boot-application-1.0.0-SNAPSHOT]
[INFO] Processing war project
[INFO] Webapp assembled in [116 msecs]
[INFO] Building war: /Users/nanlei/Dev/Workspace/first-spring-boot-application/target/first-spring-boot-application-1.0.0-SNAPSHOT.war
[INFO] 
[INFO] --- spring-boot-maven-plugin:2.3.2.RELEASE:repackage (default) @ first-spring-boot-application ---
[INFO] Replacing main artifact with repackaged archive
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  3.028 s
[INFO] Finished at: 2020-08-04T13:56:06+08:00
[INFO] ------------------------------------------------------------------------
```

Maven日志中显示了```spring-boot-maven-plugin:2.3.2.RELEASE```插件的执行信息，再次启动程序：
```cmd
$ java -jar target/first-spring-boot-application-1.0.0-SNAPSHOT.war 

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.3.2.RELEASE)

2020-08-04 13:58:03.282  INFO 4253 --- [           main] deep.in.spring.boot.App                  : Starting App on nanleis-MacBook-Pro.local with PID 4253 (/Users/nanlei/Dev/Workspace/first-spring-boot-application/target/first-spring-boot-application-1.0.0-SNAPSHOT.war started by nanlei in /Users/nanlei/Dev/Workspace/first-spring-boot-application)
2020-08-04 13:58:03.285  INFO 4253 --- [           main] deep.in.spring.boot.App                  : No active profile set, falling back to default profiles: default
2020-08-04 13:58:04.239  INFO 4253 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2020-08-04 13:58:04.251  INFO 4253 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2020-08-04 13:58:04.251  INFO 4253 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.37]
2020-08-04 13:58:04.682  INFO 4253 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2020-08-04 13:58:04.682  INFO 4253 --- [           main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 1342 ms
2020-08-04 13:58:04.843  INFO 4253 --- [           main] o.s.s.concurrent.ThreadPoolTaskExecutor  : Initializing ExecutorService 'applicationTaskExecutor'
2020-08-04 13:58:05.092  INFO 4253 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2020-08-04 13:58:05.105  INFO 4253 --- [           main] deep.in.spring.boot.App                  : Started App in 2.212 seconds (JVM running for 2.633)
```

至此项目可以正常启动了，在调整依赖配置的过程中(```<parent>```方式到```<dependencyManagement>```方式)，遇到了如下问题：

+ ```maven-war-plugin```插件在版本上的差异  
    + 2.2 版本默认的打包规则是必须存在Web应用部署描述文件```WEB-INF/web.xml```  
    + 3.2.3 版本中则调整了该行为  
  
+ ```spring-boot-maven-plugin```插件在使用时注意添加```repackage```的```goal```，否则可能不会添加Spring Boot的引导依赖，从而无法启动程序
    
+ 出于习惯，通常不将```spring-boot-dependencies```直接作为```<parent>```，尽管```spring-boot-starter-parent```也只是简单继承了```spring-boot-dependencies```

