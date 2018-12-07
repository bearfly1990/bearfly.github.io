---
layout: post
title: Java Annotation
subtitle: A brief introduction to annotation in java
date: 2018-12-06
author: BF
header-img: img/bf/lake_04.jpg
catalog: true
tags:
  - java
  - annotation
  - reflect
---

# 背景

开始回顾 Java 的一些基础知识，今天看了下注解。

像在 Spring 中，可以使用 xml 来配置类之间的关系与实例化，虽然结构清晰，但不是很方便，使用注解就能很方便的让框架自己去识别你对类的划分与使用方式。

以下面上周写的简单 REST API 为例，通过使用 Spring MVC 相关注解就能配置方法与 HTTP Request 的对应。而使用 swagger 相关的注解，便很方便的生成对应的 API 文档。

而这些都是框架、第三方库通过反射得到注解的信息之后再得到的。

```java
package fun.bearfly.swagger.controller;

import java.util.ArrayList;
import java.util.Collections;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

import fun.bearfly.swagger.model.EWord;
import io.swagger.annotations.ApiImplicitParam;
import io.swagger.annotations.ApiImplicitParams;
import io.swagger.annotations.ApiOperation;

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

# Annotation

Annotation 是代码里的特殊标记，这些标记可以在编译、类加载、运行时被读取，并执行相应的处理。Annotation 能被用来为程序元素(类、方法、成员变量等)设置元数据。通过使用 Annotation，程序开发人员可以在不改变原有逻辑的情况下，在源文件嵌入一些补充信息。Annotatio n 就像修饰符一样被使用，

如果希望让程序中的 Annotation 能在运行时起一定的作用，只有通过某种配套的工具对 Annotation 中的信息进行访问的处理，访问和处理 Annotation 的工具统称**APT (Annotation Processing Tool)**。

## 基本的 Annotation

Java 常见与使用的基本的 Annotation 有三种：

| 注解              | 作用               |
| ----------------- | ------------------ |
| @Override         | 限定重写父类的方法 |
| @Deprecated       | 标示已过时         |
| @SuppressWarnings | 抑制编译器警告     |

在后续的版本中加入了更多的注解，比如在 Java8 中有 @FunctionalInterface。

@FunctionalInterface，主要用于编译级错误检查，加上该注解，当你写的接口不符合函数式接口定义的时候，编译器会报错。

接口中包含了两个抽象方法，违反了函数式接口的定义，Eclipse 报错提示其不是函数式接口。
`Invalid '@FunctionalInterface' annotation; IFunctionalInterfaceDemo is not a functional interface`

```
 @FunctionalInterface
 interface GreetingService
 {
 	void sayMessage(String message);
 	void sayHi(String message)
 }
```

## 自定义 Annotation

我们可以自己定义注解，比如：

```java
@Target(ElementType.METHOD)
public @interface Login {
    String username() default "bearfly1990";
    String password() default "123456";
}
```

可以看到在定义注解的时候我们又用到了别的注解，这些注解是用来注解注解的，即元注解:)

四大元注解：

- @Target：注解能用在哪儿
  - @Target(ElementType.METHOD)
    - ElementType 可能的值：
    - TYPE 用于 class 定义
    - CONSTRUCTOR 用于构造方法
    - METHOD 用于方法
    - FIELD 用于成员变量
    - LOCAL_VARIABLE 局部变量声明
    - PACKAGE 包声明
    - PARAMETER 方法参数声明
    - ANNOTATION_TYPE
    - TYPE_PARAMETER
    - TYPE_USE
- @Retention：注解信息在哪个阶段有效
  - @Retention(RetentionPolicy.RUNTIME)
  - RetentionPolicy 可能的值：
  - SOURCE：源码阶段，编译时，一般是用来告诉编译器一些信息，将被编译器丢弃
  - CLASS：注解在 class 文件中可用，会被 VM 丢弃
  - **RUNTIME：运行时，VM 在运行时也保留注解，这个才能通过反射获取到**
- @ Documented
  - 将此注解包含在 JavaDoc 中
- @ Inherited
  - 允许子类继承父类中的注解

下面建了另外一个简单的例子,使用到了@Repeatable (since java8)

```java
@Repeatable(TestCases.class)
@Retention(RetentionPolicy.RUNTIME)
public @interface TestCase {
	public int id();
	public String description() default "";
}
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface TestCases {
	TestCase[] value();
}
```

下面是使用的方式：

```java
public class TestCaseDemo {
    @TestCase(id = 1, description = "test username")
    public boolean testUsername(String username){
        return false;
    }

	@TestCase(id = 2, description = "test password")
    public boolean testPassword(String password){
        return false;
    }

	@TestCase(id = 3, description = "test username and password")
    @TestCase(id = 4, description = "test login")
    public boolean testLogin(String username, String password){
        return false;
    }
}

```

## 取得注解信息

通过**反射**我们可以得到 Annotation 的信息，下面是例子：

```java
public class AnnotationDemo {
	public static void main(String[] args) {
		Method[] methods = TestCaseDemo.class.getMethods();
		for (Method method : methods) {
			Annotation[] annotations = method.getAnnotations();
			for (Annotation annotation : annotations) {
				if (annotation != null && annotation instanceof TestCase) {
					System.out.println(((TestCase) annotation).id() + " " + ((TestCase) annotation).description());
				} else if (annotation != null && annotation instanceof TestCases) {
					TestCase[] repeatTestCases = method.getAnnotationsByType(TestCase.class);
					for (TestCase testCase : repeatTestCases) {
						System.out.println(testCase.id() + " " + testCase.description());
					}
				}
			}
		}
	}
}
```

#最后

更多的细节待续...

---

参考：

- [Java 中 Annotation 用法](https://www.cnblogs.com/be-forward-to-help-others/p/6846821.html)
- [Java 9：装 B 之前你必须要会的——泛型，注解，反射](https://blog.csdn.net/cowthan/article/details/53575326)
