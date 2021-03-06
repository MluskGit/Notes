`目录 start`
 
- [SpringBoot](#springboot)
    - [开始使用](#开始使用)
    - [安装SpringBootCLI](#安装springbootcli)
    - [Springboot的测试模块](#springboot的测试模块)
    - [参考博客](#参考博客)
    - [配置文件](#配置文件)
        - [多种配置文件并切换](#多种配置文件并切换)
            - [yml方式](#yml方式)
            - [yml和Properties结合](#yml和properties结合)
    - [Web模块](#web模块)
        - [上传下载文件](#上传下载文件)
        - [错误页面跳转配置](#错误页面跳转配置)
        - [跨域](#跨域)
    - [HTTPS的配置](#https的配置)
    - [线程池](#线程池)
    - [项目部署](#项目部署)
        - [生成指定文件](#生成指定文件)
            - [war](#war)
            - [jar](#jar)
        - [构建docker镜像](#构建docker镜像)
            - [gradle结合docker](#gradle结合docker)

`目录 end` *目录创建于2018-01-14*
****************************************
# SpringBoot
> 一个简化Spring开发的框架,微服务SpringCloud的基础

## 开始使用
> 使用Idea的话就可以直接创建项目 使用别的可以下载zip进行导入 [官方下载地址](http://start.spring.io/)

## 安装SpringBootCLI
- 安装SDKMAN
    - 使用git bash运行  `curl -s get.sdkman.io | bash`
    - `source "/Users/{yourname}/.sdkman/bin/sdkman-init.sh" `根据实际目录去运行
    - spring --version

- *注意：所有`Controller`类要和`*Application`类 *同包或子包*
    
## Springboot的测试模块
- 可以使用MockMvc来测试Controller层的代码
- 可以使用MockMvc的SpringSecurity支持来测试安全模块
- 使用 WebIntegraionTest 测试运行中的Web容器
	- 启动嵌入式的Servlet容器来进行测试，下断言
- 使用随机端口启动服务器 配置local.server.port=0
- 使用Selenium来测试HTML页面，模拟浏览器的动作，查看系统运行状态

**********
## 参考博客
- [Springboot探索](https://juejin.im/post/598dd709f265da3e213f0c57)
- [SpringBoot入门](http://blog.csdn.net/jsyxcjw/article/details/46763639)

***********
## 配置文件

> 配置文件(`application.properties或者yml`)加载顺序 [官方文档说明](https://docs.spring.io/spring-boot/docs/1.5.9.RELEASE/reference/htmlsingle/#boot-features-external-config-application-property-files)
>> 1.当前Jar/War目录下的/config目录 `file:./config/`  
>> 2.当前目录 `file:./`  
>> 3.classpath 里的/config目录 `classpath:/config/`  
>> 4.classpath 根目录 `classpath:/`  

> 自定义配置文件名就要运行时加参数  
>> `java -jar myproject.jar --spring.config.name=myproject`  
>> `java -jar myproject.jar --spring.config.location=classpath:/default.properties,classpath:/override.properties` <br/>

- [配置文件的使用](http://www.itwendao.com/article/detail/391009.html)
- [Spring boot配置文件 application.properties](https://www.tuicool.com/articles/veUjQba)
- [SpringBoot常用配置](https://my.oschina.net/wangnian/blog/666641)
- [使用Gradle整合SpringBoot+Vue.js-开发调试与打包](https://segmentfault.com/a/1190000008968295)
- [配置文件加密](https://yq.aliyun.com/articles/182720)

- [自定义配置文件](http://www.cnblogs.com/java-zhao/p/5542154.html)`将应用配置外置并注入成bean`
- [配置文件外置](http://www.cnblogs.com/xiaoqi/p/6955288.html)

### 多种配置文件并切换
#### yml方式
- 单文件`配置文件 application.yml`
```yml
	spring:
      profiles:
        active: development # 选用开发模式
    ---
    spring:
      profiles: development
      //一系列配置
    ---
    spring:
      profiles: production
      //一系列配置
```
- 或者 多文件放 `application-{profile}.yml`

#### yml和Properties结合
- 格式：`application-{profile}.properties`
- 将上面的开发部分，发行部分的配置创建两个配置文件 `application-dev.properties` 和 `application-prod.properties`
- 在主配置文件`application.yml`中指明
```yml
    spring:
      profiles:
        active: dev或者是prod
```

************
## Web模块
### 上传下载文件
> 第一种直接上传到应用的webroot或者resources目录下，第二种上传到数据库中，第三种使用ftp。

- [Springboot上传文件](http://www.cnblogs.com/studyCenter/p/6665171.html)
- 上传文件有大小限制，使用如下方法进行配置 [参考博客](http://makaidong.com/studyDetail/11882_45833.html)
```java
@Bean
public MultipartConfigElement multipartConfigElement() {
    MultipartConfigFactory factory = new MultipartConfigFactory();
    //单个文件最大
    factory.setMaxFileSize("80MB"); //KB,MB
    // 设置总上传数据总大小
    factory.setMaxRequestSize("102400KB");
    return factory.createMultipartConfig();
}
```
### 错误页面跳转配置
```java
@Configuration
public class MvcConfig extends WebMvcConfigurerAdapter {
    @Bean
    public EmbeddedServletContainerCustomizer containerCustomizer() {
        return (container -> {
            ErrorPage error401Page = new ErrorPage(HttpStatus.FORBIDDEN, "/403.html");
            ErrorPage error404Page = new ErrorPage(HttpStatus.NOT_FOUND, "/404.html");
            ErrorPage error500Page = new ErrorPage(HttpStatus.INTERNAL_SERVER_ERROR, "/500.html");
            container.addErrorPages(error401Page, error404Page, error500Page);
        });
    }
}
```
### 跨域
> 不同的域名（主机）端口都会导致跨域问题

```java
@Configuration
public class CorsConfig {
    private CorsConfiguration buildConfig() {
        CorsConfiguration corsConfiguration = new CorsConfiguration();
        corsConfiguration.addAllowedOrigin("*"); // 允许任何域名使用
        corsConfiguration.addAllowedHeader("*"); // 允许任何头
        corsConfiguration.addAllowedMethod("*"); // 允许任何方法（post、get等）
        return corsConfiguration;
    }
    @Bean
    public CorsFilter corsFilter() {
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", buildConfig()); // 4
        return new CorsFilter(source);
    }
}
```


*******************
## HTTPS的配置
> [参考博客](https://www.drissamri.be/blog/java/enable-https-in-spring-boot/)

- 签发证书：
```sh
keytool -genkey -alias tomcat -storetype PKCS12 -keyalg RSA -keysize 2048 -keystore keystore.p12 -validity 3650
```
```yml
server:
  context-path: /myth
  ssl:
    key-store: classpath:keystore.p12
    key-store-password: demo1429336
    key-store-type: PKCS12
    key-alias: tomcat
  port: 8888
  session:
    timeout: 3000
```
`任意的一个@Configuration注解类里添加`
```java
@Bean
public TomcatEmbeddedServletContainerFactory servletContainerFactory() {
    TomcatEmbeddedServletContainerFactory factory =
        new TomcatEmbeddedServletContainerFactory() {
            @Override
            protected void postProcessContext(Context context) {
                //SecurityConstraint必须存在，可以通过其为不同的URL设置不同的重定向策略。
                SecurityConstraint securityConstraint = new SecurityConstraint();
                securityConstraint.setUserConstraint("CONFIDENTIAL");
                SecurityCollection collection = new SecurityCollection();
                collection.addPattern("/*");
                securityConstraint.addCollection(collection);
                context.addConstraint(securityConstraint);
            }
        };
    factory.addAdditionalTomcatConnectors(createHttpConnector());
    return factory;
}

private Connector createHttpConnector() {
    Connector connector = new Connector("org.apache.coyote.http11.Http11NioProtocol");
    connector.setScheme("http");
    connector.setSecure(false);
    connector.setPort(8887);//http端口（这是要新增加的一个端口）
    connector.setRedirectPort(8888);// https 端口配置文件中tomcat启动的默认端口
    return connector;
}
```

- 另一种方式 [参考博客](http://www.cnblogs.com/xinzhao/p/4950689.html)
`方式不一样，没有成功`
```sh
    ############ 证书颁发机构
    # CA机构私钥
    openssl genrsa -out ca.key 2048
    # CA证书
    openssl req -x509 -new -key ca.key -out ca.crt
    ############ 服务端
    # 生成服务端私钥
    openssl genrsa -out server.key 2048
    # 生成服务端证书请求文件
    openssl req -new -key server.key -out server.csr
    # 使用CA证书生成服务端证书  关于sha256，默认使用的是sha1，在新版本的chrome中会被认为是不安全的，因为使用了过时的加密算法。
    openssl x509 -req -sha256 -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -days 3650 -out server.crt    
    # 打包服务端的资料为pkcs12格式(非必要，只是换一种格式存储上一步生成的证书) 生成过程中，需要创建访问密码，请记录下来。
    openssl pkcs12 -export -in server.crt -inkey server.key -out server.pkcs12
    # 生成服务端的keystore（.jks文件, 非必要，Java程序通常使用该格式的证书） 生成过程中，需要创建访问密码，请记录下来。
    keytool -importkeystore -srckeystore server.pkcs12 -destkeystore server.jks -srcstoretype pkcs12
    # 把ca证书放到keystore中（非必要）
    keytool -importcert -keystore server.jks -file ca.crt
```

****************
## 线程池
- [参考博客](https://hacpai.com/article/1501152977477?p=1&m=0)
- 多线程以及异常处理 [参考博客](http://www.liuhaihua.cn/archives/496733.html)
    - 因为多线程的特性，所以异常只能在子线程中处理不能抛出到主线程里，但是 Spring实现的线程池可以返回一个异常信息对象

***************************************
## 项目部署
### 生成指定文件
#### war
- 部署为war必须的类，一般在创建项目时选war就会自动生成，选jar就要手动添加
```java
    public class ServletInitializer extends SpringBootServletInitializer {
        @Override
        protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
            return application.sources(DemoApplication.class);
        }
    }
```
- maven： `mvn war` 即可 mvn package -DskipTests
- gradle: `gradle war` 然后 `gradle bootRepackage` 即可

#### jar
- 没有特殊的配置，打包即用
    - maven: `mvn package` 即可生成可执行的jar
    - gradle:`gradle jar` 然后 `gradle bootRepackage` 也生成可执行jar

### 构建docker镜像
> 方便监控应用状态，cpu 内存 流量

- 先构建得到war或jar，然后根据dockerfile构建一个镜像
```Dockerfile
FROM frolvlad/alpine-oraclejdk8:slim
ADD weixin-1.0.0.war app.war
ENTRYPOINT ["java","-jar","/app.war"]
```

#### gradle结合docker
`build.gradle`
```groovy
apply plugin: 'docker'
task buildDocker(type: Docker, dependsOn: build) {
    push = true
    applicationName = jar.baseName
    dockerfile = file('src/main/docker/Dockerfile')
    doFirst {
        copy {
            from war
            into stageDir
        }
    }
}
```
`Dockerfile`
```Dockerfile
FROM frolvlad/alpine-oraclejdk8:slim
VOLUME /tmp
ADD weixin-1.0.0.war app.war
ENTRYPOINT ["java","-jar","/app.war"]
```