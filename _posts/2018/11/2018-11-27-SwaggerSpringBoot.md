---
layout: post
title: Swagger in SpringBoot
subtitle: Generate RESTful API documentation by swagger and springboot
date: 2018-11-27
author: BF
header-img: img/bf/night_01.jpg
catalog: true
tags:
  - swagger
  - springboot
  - java
  - RESTful
---

# 背景

前两天提到项目组打算用 swagger 来做为 RESTful API 的解决方案，现在已经确定下来了。

之前看到有文章介绍 SpringBoot 中结合 Swagger 直接生成文档的方法，今天尝试了一下，入门还是比较简单的。

# 建立 Maven 工程

我们使用 Maven 管理 jar 包依赖，网上有许多的教程，就不赘述。今天在配置环境的时候，看到有一个阿里云的镜像，可以记录一下：

```xml
<mirror>
    <id>alimaven</id>
    <name>aliyun maven</name>
    <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
    <mirrorOf>central</mirrorOf>
</mirror>
```

## 引入相关依赖

完整的配置请看[pom.xml](https://github.com/bearfly1990/PowerScript/blob/master/Java/Swagger/pom.xml)

### SpringBoot 相关

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.0.RELEASE</version>
    <relativePath /> <!-- lookup parent from repository -->
</parent>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <optional>true</optional>
    </dependency>
</dependencies>
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <fork>true</fork>
            </configuration>
        </plugin>
    </plugins>
</build>
```

### Swagger 相关依赖

```xml
<!-- swagger2 -->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.2.2</version>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.2.2</version>
</dependency>
```

### 热启动

通过下面的片段来开启了热启动，代码更新后自动重启，方便很多。

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <optional>true</optional>
</dependency>
```

```xml
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <configuration
        <fork>true</fork>
    </configuration>
</plugin>
```

# 创建 Swagger2 配置类

在 SpringBoot 主类同级目录下新建 Swagger 的配置类如下：

```java
@Configuration
@EnableSwagger2
public class Swagger2 {

    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.basePackage("fun.bearfly"))
                .paths(PathSelectors.any())
                .build();
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("Generate RESTful APIs with Spring Boot and Swagger2 ")
                .description("More info please see:https://bearfly1990.github.io/")
                .termsOfServiceUrl("https://bearfly1990.github.io/")
                .contact("bearfly1990")
                .version("1.0")
                .build();
    }

}
```

# Controller 中的注解

接口文档的信息通过注解来写入，所以就放在 Controller 这一层。

下面的代码主要模拟了对单词的 CRUD 操作，每个接口都有对应的接口注释，通过@ApiOperation， @ApiImplicitParam 等注解来实现，这个我之前的文章有提到：[Swagger#代码中的使用](https://bearfly1990.github.io/2018/11/25/Swagger/#代码中的使用)

```java
@RestController
@RequestMapping(value="/ewords")
public class EWordController {
	static Map<Long, EWord> ewordsMap = Collections.synchronizedMap(new HashMap<Long, EWord>());

	@ApiOperation(value="HelloWorld", notes="")
	@RequestMapping(value="helloworld", method=RequestMethod.GET)
	public String sayHello() {
		return "hello, world!";
	}

    @ApiOperation(value="GetEword", notes="")
    @RequestMapping(value={""}, method=RequestMethod.GET)
    public List<EWord> getEWordList() {
        List<EWord> ewords = new ArrayList<EWord>(ewordsMap.values());
        return ewords;
    }

    @ApiOperation(value="CreateEword", notes="Create Eword by Eword")
    @ApiImplicitParam(name = "eword", value = "user domain", required = true, dataType = "EWord")
    @RequestMapping(value="", method=RequestMethod.POST)
    public String postEWord(@RequestBody EWord eword) {
    	ewordsMap.put(eword.getId(), eword);
        return "success";
    }

    @ApiOperation(value="GetEwordDetails", notes="Get eword by id")
    @ApiImplicitParam(name = "id", value = "ewordID", required = true, dataType = "Long")
    @RequestMapping(value="/{id}", method=RequestMethod.GET)
    public EWord getEWord(@PathVariable Long id) {
        return ewordsMap.get(id);
    }

    @ApiOperation(value="UpdateEwordInfo", notes="Update eword by id")
    @ApiImplicitParams({
            @ApiImplicitParam(name = "id", value = "ewordID", required = true, dataType = "Long"),
            @ApiImplicitParam(name = "user", value = "ewordDomain", required = true, dataType = "EWord")
    })
    @RequestMapping(value="/{id}", method=RequestMethod.PUT)
    public String putEWord(@PathVariable Long id, @RequestBody EWord newUser) {
        EWord eword = ewordsMap.get(id);
        eword.setCnWord(newUser.getCnWord());
        eword.setEnWord(newUser.getEnWord());
        ewordsMap.put(id, eword);
        return "success";
    }

    @ApiOperation(value="DeleteEword", notes="Delete eword by id")
    @ApiImplicitParam(name = "id", value = "ewordID", required = true, dataType = "Long")
    @RequestMapping(value="/{id}", method=RequestMethod.DELETE)
    public String deleteEWord(@PathVariable Long id) {
        ewordsMap.remove(id);
        return "success";
    }
}
```

# 生成文档

上面的配置与代码完成后，直接启动 SpringBoot，访问：http://localhost:8080/swagger-ui.html 就能查看文档了。

![swagger-ui](/img/post/2018/11/2018-11-27-SwaggerSpringBoot.jpg)

搞定~

工程目录：https://github.com/bearfly1990/PowerScript/tree/master/Java/Swagger

---

参考：

- [深入理解 RPC : 基于 Python 自建分布式高并发 RPC 服务](https://www.jianshu.com/p/8033ef83a8ed) - 程序猿 DD
- [构建微服务：Spring boot 入门篇](https://www.cnblogs.com/ityouknow/p/5662753.html) - 微信公众号：纯洁的微笑
