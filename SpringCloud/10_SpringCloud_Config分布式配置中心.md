---
title: 10_SpringCloud_Config分布式配置中心
date: 2019-12-08 21:13:32
tags: 
 - SpringCloud
categories:
 - SpringCloud
---

# 10_SpringCloud_Config分布式配置中心

## SpringCloud Config配置中心概述

### 分布式系统面临的配置问题

微服务意味着要将单个应用中的业务拆分成一个一个子服务，每个服务的粒度相对较小，因此系统中会出现大量的服务。由于每个服务都需要必要的配置信息才能运行，所以一套集中式的、动态的配置管理设施是必不可少的。

SpringCloud提供了Config Server来解决这个问题，我们每一个微服务自己带着一个application.yaml，上百个配置文件的管理......



### SpringCloud Config是什么？

![image-20191208212304136](10_SpringCloud_Config%E5%88%86%E5%B8%83%E5%BC%8F%E9%85%8D%E7%BD%AE%E4%B8%AD%E5%BF%83/image-20191208212304136.png)

SpringCloud Config为微服务架构中的微服务提供集中化的**外部配置支持**，配置服务器为**各个不同的微服务应用**的所有环境提供了一个**中心化的外部配置**；



### SpringCloud Config怎么玩？

SpringCloud Config分为**服务端和客户端两部分**；

- 服务端：

  服务端也称为**分布式配置中心，它是一个独立的微服务应用**，用于连接配置服务器并且为客户端提供获取配置信息，加密/解密信息等访问接口；

- 客户端：

  客户端则通过指定的配置中心来管理应用资源，以及业务相关的配置内容，并在启动的时候从配置中心获取和加载配置信息；

  配置服务器默认采用git来存储配置信息，这样就有助于对环境配置进行版本管理，并且可以通过git客户端工具来方便的管理和访问配置内容；





### SpringCloud Config能干嘛？

1. 集中管理配置文件

2. 不同环境不同配置，动态化的配置更新，分环境部署，比如dev/test/prod/beta/release
3. 运行期间动态调整配置，不再需要在每个服务部署的机器上编写配置文件，服务会向配置中心统一拉取配置自己的信息
4. 当配置发生变动的时候，服务不需要重新启动即可感知配置的变化并应用新的配置
5. 将配置信息已REST接口的形式暴露



### SpringCloud Config与GitHub整合配置

由于SpringCloud Config默认使用的是Git来存储配置文件（也有其他的方式，比如支持SVN和本地文件），但最推荐的还是Git，而且使用的是http/https访问的形式；



## SpringCloud Config服务端配置

1. 用自己的GitHub账号在GitHub上新建一个名为microservicecloud-config的新仓库

2. 由上一步获得SSH协议的git地址在本地硬盘目录上新建git仓库并clone

   - `git clone git@github.com:tomxwd/microservicecloud-config.git`

3. 在本地的microservicecloud-config里面新建一个application.yaml

   **必须要用UTF-8的形式保存！！！**

   ```yaml
   # 切记保存为UTF-8形式
   spring:
     profiles:
       active:
         -dev
   ---
   spring:
     profiles: dev     # 开发环境
     application:
       name: microservicecloud-config-tomxwd-dev
   ---
   spring:
     profiles: test    # 测试环境
     application:
       name: microservicecloud-config-tomxwd-test
   ```

4. 将上一步的yaml文件推送到github上

   `git add .`

   `git commit -m "提交配置文件"`

   `git push origin master`

5. 新建Module模块microservicecloud-config-3344，它就是Cloud的配置中心模块

6. pom.xml

   - 修改部分：

     ```xml
     <dependency>
         <groupId>org.springframework.cloud</groupId>
         <artifactId>spring-cloud-config-server</artifactId>
     </dependency>
     ```

     如果报错就加：

     ```xml
     <!-- 报错就加这个依赖 -->
     <dependency>
         <groupId>org.eclipse.jgit</groupId>
         <artifactId>org.eclipse.jgit</artifactId>
         <version>4.6.0.201612231935-r</version>
     </dependency>
     ```

   - 完整内容：

     ```xml
     <?xml version="1.0" encoding="UTF-8"?>
     <project xmlns="http://maven.apache.org/POM/4.0.0"
              xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
              xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
         <parent>
             <artifactId>microservicecloud</artifactId>
             <groupId>top.tomxwd</groupId>
             <version>1.0-SNAPSHOT</version>
         </parent>
         <modelVersion>4.0.0</modelVersion>
     
         <artifactId>microservicecloud-config-3344</artifactId>
     
         <dependencies>
             <!-- springcloud-config -->
             <dependency>
                 <groupId>org.springframework.cloud</groupId>
                 <artifactId>spring-cloud-config-server</artifactId>
             </dependency>
             <!-- 将微服务provider注册进Eureka -->
             <!--        <dependency>-->
             <!--            <groupId>org.springframework.cloud</groupId>-->
             <!--            <artifactId>spring-cloud-starter-eureka</artifactId>-->
             <!--        </dependency>-->
             <!-- 报错就加这个依赖 -->
             <!--        <dependency>-->
             <!--            <groupId>org.eclipse.jgit</groupId>-->
             <!--            <artifactId>org.eclipse.jgit</artifactId>-->
             <!--            <version>4.6.0.201612231935-r</version>-->
             <!--        </dependency>-->
             <!-- 图形化监控 -->
             <dependency>
                 <groupId>org.springframework.boot</groupId>
                 <artifactId>spring-boot-starter-actuator</artifactId>
             </dependency>
             <!-- 熔断 -->
             <dependency>
                 <groupId>org.springframework.cloud</groupId>
                 <artifactId>spring-cloud-starter-hystrix</artifactId>
             </dependency>
             <!-- 内嵌jetty容器 -->
             <dependency>
                 <groupId>org.springframework.boot</groupId>
                 <artifactId>spring-boot-starter-jetty</artifactId>
             </dependency>
             <dependency>
                 <groupId>org.springframework.boot</groupId>
                 <artifactId>spring-boot-starter-web</artifactId>
             </dependency>
             <dependency>
                 <groupId>org.springframework.boot</groupId>
                 <artifactId>spring-boot-starter-test</artifactId>
             </dependency>
             <!-- 热部署 -->
             <dependency>
                 <groupId>org.springframework</groupId>
                 <artifactId>springloaded</artifactId>
             </dependency>
             <dependency>
                 <groupId>org.springframework.boot</groupId>
                 <artifactId>spring-boot-devtools</artifactId>
             </dependency>
         </dependencies>
     
     </project>
     ```

7. application.yaml

   ```yaml
   server:
     port: 3344
   spring:
     application:
       name: microservicecloud-config
     cloud:
       config:
         server:
           git:
   #          uri: git@github.com:tomxwd/microservicecloud-config.git
             uri: https://github.com/tomxwd/microservicecloud-config.git
             default-label: master
             search-paths: /
   #          ignoreLocalSshSettings: true
   ##          hostKey: someHostKey
   ##          hostKeyAlgorithm: ssh-rsa
   #          privateKey:
   ```

   如果报git clone错误就用https的形式填写GitHub仓库地址；

8. 主启动类Config_3344_StartSpringCloudApp

   ```java
   @SpringBootApplication
   @EnableConfigServer
   public class Config_3344_StartSpringCloudApp {
   
       public static void main(String[] args) {
           SpringApplication.run(Config_3344_StartSpringCloudApp.class,args);
       }
   
   }
   ```

9. windows下修改hosts文件，增加映射

   - `127.0.0.1	config-3344.com`

10. 测试通过Config微服务是否可以从GitHub上获取配置内容

    - 启动微服务3344
    - 访问：http://config-3344.com:3344/application-dev.yaml，或者yml
    - 访问：http://config-3344.com:3344/application-test.yaml，或者yml
    - 访问：http://config-3344.com:3344/application-xxx.yaml，或者yml
      - 得到的是默认的

11. 配置读取规则

    - 官网
    - /{application}-{profile}.yaml
      - http://config-3344.com:3344/application-dev.yaml
      - http://config-3344.com:3344/application-test.yaml
      - http://config-3344.com:3344/application-xxx.yaml（不存在的配置）
    - /{application}/{profile}[/{label}]
      - http://config-3344.com:3344/application/dev/master
      - http://config-3344.com:3344/application/test/master
      - http://config-3344.com:3344/application/xxx/master
    - /{label}/{application}-{profile}.yaml
      - http://config-3344.com:3344/master/application-dev.yaml
      - http://config-3344.com:3344/master/application-test.yaml
    - /{application}-{profile}.properties
    - /{label}/{application}-{profile}.properties

12. 成功实现了用SpringCloud Config通过GitHub的方式获取配置信息



## SpringCloud Config客户端配置与测试

1. 在本地microservicecloud-config仓库路径下新建文件microservicecloud-config-client.yaml

2. microservicecloud-config-client.yaml内容

   ```yaml
   spring:
     profiles:
       active:
         - dev
   
   ---
   server:
     port: 8201
   spring:
     profiles: dev
     application:
       name: microservice-config-client
   eureka:
     client:
       service-url: http://eureka-dev.com:7001/eureka/
   ---
   server:
     port: 8202
   spring:
     profiles: test
     application:
       name: microservice-config-client
   eureka:
     client:
       service-url: http://eureka-test.com:7001/eureka/
   ```

3. 将上一步的文件推送到GitHub中；

4. 新建microservicecloud-config-client-3355

5. pom.xml

   - 修改部分：

     ```xml
     <!-- SpringCloud Config 客户端 Client -->
     <dependency>
         <groupId>org.springframework.cloud</groupId>
         <artifactId>spring-cloud-starter-config</artifactId>
     </dependency>
     ```

   - 完整内容：

     ```xml
     <?xml version="1.0" encoding="UTF-8"?>
     <project xmlns="http://maven.apache.org/POM/4.0.0"
              xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
              xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
         <parent>
             <artifactId>microservicecloud</artifactId>
             <groupId>top.tomxwd</groupId>
             <version>1.0-SNAPSHOT</version>
         </parent>
         <modelVersion>4.0.0</modelVersion>
     
         <artifactId>microservicecloud-config-client-3355</artifactId>
     
         <dependencies>
             <!-- SpringCloud Config 客户端 Client -->
             <dependency>
                 <groupId>org.springframework.cloud</groupId>
                 <artifactId>spring-cloud-starter-config</artifactId>
             </dependency>
              <!--将微服务provider注册进Eureka -->
             <dependency>
                 <groupId>org.springframework.cloud</groupId>
                 <artifactId>spring-cloud-starter-eureka</artifactId>
             </dependency>
             <!-- 图形化监控 -->
             <dependency>
                 <groupId>org.springframework.boot</groupId>
                 <artifactId>spring-boot-starter-actuator</artifactId>
             </dependency>
             <!-- 熔断 -->
             <dependency>
                 <groupId>org.springframework.cloud</groupId>
                 <artifactId>spring-cloud-starter-hystrix</artifactId>
             </dependency>
             <!-- 内嵌jetty容器 -->
             <dependency>
                 <groupId>org.springframework.boot</groupId>
                 <artifactId>spring-boot-starter-jetty</artifactId>
             </dependency>
             <dependency>
                 <groupId>org.springframework.boot</groupId>
                 <artifactId>spring-boot-starter-web</artifactId>
             </dependency>
             <dependency>
                 <groupId>org.springframework.boot</groupId>
                 <artifactId>spring-boot-starter-test</artifactId>
             </dependency>
             <!-- 热部署 -->
             <dependency>
                 <groupId>org.springframework</groupId>
                 <artifactId>springloaded</artifactId>
             </dependency>
             <dependency>
                 <groupId>org.springframework.boot</groupId>
                 <artifactId>spring-boot-devtools</artifactId>
             </dependency>
         </dependencies>
     
     </project>
     ```

6. **bootstrap.yaml**

   - 是什么？

     application.yaml是用户级的资源配置项

     bootstrap.yaml是系统级的，**优先级更高**

     SpringCloud会创建一个“Bootstrap Context”，作为Spring应用的“Application Context”的**父上下文**。在初始化的时候，“Bootstrap Context”负责从外部源加载配置属性并解析配置。这两个上下文共享一个从外部获取的“Environment”。“Bootstrap”属性拥有更高的优先级，默认情况下，它们不会被本地配置覆盖。

     “Bootstrap Context”和“Application Context”有着不同的约定，所以增加一个“bootstrap.yaml”文件，保证“Bootstrap Context”和“Application Context”配置的分离；

   - 内容：

     ```yaml
     spring:
       cloud:
         config:
           name: microservicecloud-config-client # 需要从github上面读取的资源名称，注意没有yaml后缀名
           profile: dev                          # 本次访问的配置项
           label: master
           uri: http://config-3344.com:3344          # 本微服务启动后先去3344服务，通过SpringCloudConfig获取Github的服务地址
     ```

7. application.yaml

   ```yaml
   spring:
     application:
       name: microservicecloud-config-client
   ```

8. windows下修改hosts文件，增加映射

   `127.0.0.1 client-config.com`

9. 新建rest类，验证是否能够从GitHub上读取配置

   ```java
   @RestController
   public class ConfigClientRest {
   
       @Value("${spring.application.name}")
       private String applicationName;
   
       @Value("${eureka.client.service-url}")
       private String eurekaServers;
   
       @Value("${server.port}")
       private String port;
   
       @RequestMapping("/config")
       public String getConfig() {
           String str = "applicationName: " + applicationName + "\t eurekaServers: " + eurekaServers + "\t port: " + port;
           System.out.println("*****************" + str);
           return str;
       }
   
   }
   ```

10. 主启动类ConfigClient_3355_StartSpringCloudApp

    ```java
    @SpringBootApplication
    public class ConfigClient_3355_StartSpringCloudApp {
        public static void main(String[] args) {
            SpringApplication.run(ConfigClient_3355_StartSpringCloudApp.class, args);
        }
    }
    ```

11. 测试

    - 启动Config配置中心3344微服务并自测
      - http://config-3344.com:3344/application-dev.yaml
    - 启动3355作为Client准备访问
    - boostrap.yaml里面的profile值是什么，就决定了服务从git上读到什么
      - 假如目前是profile:dev
        - dev默认在GitHub上对应的端口是8201
        - http://client-config.com:8201/config
      - 假如目前是profile:test
        - test默认在GitHub上对应的端口是8202
        - http://client-config.com:8202/config
    - 可以试试修改bootstrap里面profile的值，然后热部署项目试试；

12. 成功实现了客户端3355访问SpringCloud Config3344通过GitHub获取配置信息



## SpringCloud Config配置实战

目前的情况：

1. Config服务端配置OK且测试通过，我们可以和Config以及GitHub进行配置修改并获得内容；
2. 此时我们做一个Eureka以及一个Dept访问的微服务，将两个微服务的配置统一交给GitHub，实现统一配置分布式管理，完成多环境的变更；



步骤：

1. GitHub配置文件本地配置

   - 在microservicecloud-config仓库下新建文件microservicecloud-config-eureka-client.yaml

   - microservicecloud-config-eureka-client.yaml内容：

     ```yaml
     spring:
         profiles:
             active:
                 - dev
     ---
     server:
         port: 7001              # 注册中心占用7001端口，冒号后面必须要有空格
     spring:
         profiles: dev
         application:
             name: microservicecloud-config-eureka-client
     eureka:
         instance:
             hostname: eureka7001.com
         client:
             register-with-eureka: false        # 当前的eureka-server 不注册自己到服务列表中去
             fetch-registry: false               # 不通过eureka获取注册信息
             service-url:
                 defaultZone: http://eureka7001.com:7001/eureka/
     ---
     server:
         port: 7001
     spring:
         profiles: test
         application:
             name: microservicecloud-config-eureka-client
     eureka:
         instance:
             hostname: eureka7001.com
         client:
             register-with-eureka: false
             fetch-registry: false
             service-url:
                 defaultZone: http://eureka7001.com:7001/eureka/
     ```

   - 在microservicecloud-config仓库下新建文件microservicecloud-config-dept-client.yaml

   - microservicecloud-config-dept-client.yaml内容：

     ```yaml
     spring:
         profiles:
             active:
                 - dev
     ---
     server:
         port: 8001
     spring:
         profiles: dev
         application:
             name: microservicecloud-config-dept-client
         datasourece:
             type: com.alibaba.druid.pool.DruidDataSource
             driver-class-name: org.gjt.mm.mysql.Driver
             url: jdbc:mysql://120.25.120.125:12350/cloudDB01?useUnicode=true&characterEncoding=utf8        # 数据库名称
             username: root
             password: root
             dbcp2:
                 min-idle: 5
                 initial-size: 5
                 max-total: 5
                 max-wait-millis: 200
     mybatis:
         config-location: classpath:mybatis/mybatis.cfg.xml
         mapper-locations:
             - classpath: mybatis/mapper/**/*.xml
     eureka:
         client:
             service-url:
                 defaultZone: http://eureka7001.com:7001/eureka/
         instance:
             instance-id: dept-8001.com
             prefer-ip-address: true
     info:
         app.name: tomxwd-microservicecloud
         company.name: www.tomxwd.top
         build.artifactId: $project.artifactId$
         bulid.version: $project.version$
     ---
     server:
         port: 8001
     spring:
         profiles: test
         application:
             name: microservicecloud-config-dept-client
         datasourece:
             type: com.alibaba.druid.pool.DruidDataSource
             driver-class-name: org.gjt.mm.mysql.Driver
             url: jdbc:mysql://120.25.120.125:12350/cloudDB02?useUnicode=true&characterEncoding=utf8        # 数据库名称
             username: root
             password: root
             dbcp2:
                 min-idle: 5
                 initial-size: 5
                 max-total: 5
                 max-wait-millis: 200
     mybatis:
         config-location: classpath:mybatis/mybatis.cfg.xml
         mapper-locations:
             - classpath: mybatis/mapper/**/*.xml
     eureka:
         client:
             service-url:
                 defaultZone: http://eureka7001.com:7001/eureka/
         instance:
             instance-id: dept-8001.com
             prefer-ip-address: true
     info:
         app.name: tomxwd-microservicecloud
         company.name: www.tomxwd.top
         build.artifactId: $project.artifactId$
         bulid.version: $project.version$
     ```

2. Config版本的Eureka服务端

   - 新建工程microservicecloud-config-eureka-client-7001

   - pom.xml

     ```xml
     <?xml version="1.0" encoding="UTF-8"?>
     <project xmlns="http://maven.apache.org/POM/4.0.0"
              xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
              xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
         <parent>
             <artifactId>microservicecloud</artifactId>
             <groupId>top.tomxwd</groupId>
             <version>1.0-SNAPSHOT</version>
         </parent>
         <modelVersion>4.0.0</modelVersion>
     
         <artifactId>microservicecloud-config-eureka-client-7001</artifactId>
     
         <dependencies>
             <!-- config服务端依赖 -->
             <dependency>
                 <groupId>org.springframework.cloud</groupId>
                 <artifactId>spring-cloud-starter-config</artifactId>
             </dependency>
             <!-- eureka-server服务端 -->
             <dependency>
                 <groupId>org.springframework.cloud</groupId>
                 <artifactId>spring-cloud-starter-eureka-server</artifactId>
             </dependency>
             <!-- 热部署 -->
             <dependency>
                 <groupId>org.springframework</groupId>
                 <artifactId>springloaded</artifactId>
             </dependency>
             <dependency>
                 <groupId>org.springframework.boot</groupId>
                 <artifactId>spring-boot-devtools</artifactId>
             </dependency>
         </dependencies>
     
     </project>
     ```

   - bootstrap.yaml

     ```yaml
     spring:
       cloud:
         config:
           name: microservicecloud-config-eureka-client
           profile: dev
           label: master
           uri: http://config-3344.com:3344
     ```

   - application.yaml

     ```yaml
     spring:
       application:
         name: microservicecloud-config-eureka-client
     ```

   - 主启动类Config_Git_EurekaServerApplication

     ```java
     @SpringBootApplication
     @EnableEurekaServer
     public class Config_Git_EurekaServerApplication {
     
         public static void main(String[] args) {
             SpringApplication.run(Config_Git_EurekaServerApplication.class, args);
         }
     
     }
     ```

   - 测试

     - 启动microservicecloud-config-3344微服务，保证Config总配置是ok的
       - http://config-3344.com:3344/microservice-config-eureka-client-dev.yaml
     - 再启动microservicecloud-config-eureka-client-7001微服务
       - http://eureka7001.com:7001

3. Config版本的Dept微服务

   - 参考之前的8001拷贝后新建工程microservicecloud-config-dept-client-8001

   - pom.xml

     ```xml
     <?xml version="1.0" encoding="UTF-8"?>
     <project xmlns="http://maven.apache.org/POM/4.0.0"
              xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
              xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
         <parent>
             <artifactId>microservicecloud</artifactId>
             <groupId>top.tomxwd</groupId>
             <version>1.0-SNAPSHOT</version>
         </parent>
         <modelVersion>4.0.0</modelVersion>
     
         <artifactId>microservicecloud-config-dept-client-8001</artifactId>
         <dependencies>
             <dependency>
                 <groupId>top.tomxwd</groupId>
                 <artifactId>microservicecloud-api</artifactId>
                 <version>${project.version}</version>
             </dependency>
             <dependency>
                 <groupId>junit</groupId>
                 <artifactId>junit</artifactId>
             </dependency>
             <dependency>
                 <groupId>mysql</groupId>
                 <artifactId>mysql-connector-java</artifactId>
             </dependency>
             <dependency>
                 <groupId>com.alibaba</groupId>
                 <artifactId>druid</artifactId>
             </dependency>
             <dependency>
                 <groupId>ch.qos.logback</groupId>
                 <artifactId>logback-core</artifactId>
             </dependency>
             <dependency>
                 <groupId>org.mybatis.spring.boot</groupId>
                 <artifactId>mybatis-spring-boot-starter</artifactId>
             </dependency>
             <!-- 内嵌jetty容器 -->
             <dependency>
                 <groupId>org.springframework.boot</groupId>
                 <artifactId>spring-boot-starter-jetty</artifactId>
             </dependency>
             <dependency>
                 <groupId>org.springframework.boot</groupId>
                 <artifactId>spring-boot-starter-web</artifactId>
             </dependency>
             <dependency>
                 <groupId>org.springframework.boot</groupId>
                 <artifactId>spring-boot-starter-test</artifactId>
             </dependency>
             <!-- 热部署 -->
             <dependency>
                 <groupId>org.springframework</groupId>
                 <artifactId>springloaded</artifactId>
             </dependency>
             <dependency>
                 <groupId>org.springframework.boot</groupId>
                 <artifactId>spring-boot-devtools</artifactId>
             </dependency>
             <!-- 将微服务provider注册进Eureka -->
             <dependency>
                 <groupId>org.springframework.cloud</groupId>
                 <artifactId>spring-cloud-starter-eureka</artifactId>
             </dependency>
             <dependency>
                 <groupId>org.springframework.cloud</groupId>
                 <artifactId>spring-cloud-starter-config</artifactId>
             </dependency>
             <!-- actuator监控以及信息配置 -->
             <dependency>
                 <groupId>org.springframework.boot</groupId>
                 <artifactId>spring-boot-starter-actuator</artifactId>
             </dependency>
             <!-- SpringCloud-config配置 -->
             <dependency>
                 <groupId>org.springframework.cloud</groupId>
                 <artifactId>spring-cloud-starter-config</artifactId>
             </dependency>
         </dependencies>
     </project>
     ```

   - bootstrap.yaml

     ```yaml
     spring:
       cloud:
         config:
           name: microservicecloud-config-dept-client
           profile: dev
           label: master
           uri: http://config-3344.com:3344
     ```

   - application.yaml

     ```yaml
     spring:
       application:
         name: microservicecloud-config-dept-client
     ```

   - 主启动类Config_Git_DeptProvider8001_App

     ```java
     @SpringBootApplication
     // 本服务启动后会自动注册进Eureka服务中
     @EnableEurekaClient
     // 服务发现
     @EnableDiscoveryClient
     public class Config_Git_DeptProvider8001_App {
     
         public static void main(String[] args) {
             SpringApplication.run(Config_Git_DeptProvider8001_App.class,args);
         }
     
     }
     ```

   - 配置说明

     bootstrap.yaml里面：

     - 都是以spring.cloud.config开头，然后name指定的是github上读取的资源名称，不需要后缀，profile指定环境，label表示分支名，uri表示Config的地址

   - 测试

     - 访问http://localhost:8001/dept/list，查看返回数据的数据库信息是哪个
     - 切换另一个配置，再次查看结果
     - **直接修改github上的配置（如把test环境的数据库切换为3）**，再次进行刷新









