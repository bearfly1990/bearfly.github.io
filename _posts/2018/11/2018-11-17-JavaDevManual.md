---
layout: post
title: Java Developement Manual
subtitle: 阿里巴巴Java开发手册 Effective Coding
date: 2018-11-17
author: BF
header-img: img/bf/coffee_01.jpg
catalog: true
tags:
  - java
  - manual
  - alibaba
---

# Java 开发手册(alibaba)

闲来无聊，手打一份手册，当做自己在网络的备份，方便以后查看，顺便当练习打字了，哈哈哈哈，我的五笔都生疏了，嘿嘿。

## 第一章 编程规约

本章是传统意义上的代码规范，包括变量名、代码风格、控制语句、代码注释等基本的编程习惯，以及从高并发场景中提炼出来的集合处理技巧和并发多线程的注意事项。

### 1.1 命名风格

#### 【强制】

##### 代码中的命名均不能以下画线或美元符号开始，也不能以下画线或美元符号结束。

- 代码中的命名严禁使用拼与英文混合的方式，更不允许直接使用中文的方式。
- 类名使用 UpperCamelCamel 风格， 但 DO/BO/DTO/VO/AO/PO 等情形例外。
- 方法名、参数名、成员变量 、局部变量都统一使用 lowerCamelCase 风格，必须遵从驼峰的形式。
- 常量命名全部大写，单词间用下画线隔开，力求语义表达完整清楚，不要嫌名字长。

```
e.g. MAX_STOCK_COUNT / PRIZE_NUMBER_EVERYDAY
```

- 抽象类名使用 Abstract 或 Base 开头；异常类使用 Exception 结尾；测试类命名要以它测试的类名开始，以 Test 结尾。
- 类型与中括号之间无空格相连定义数组。

```
e.g. int[] arrayDemo;
```

- POJO 类中布尔类型的变量都不要加 is 前缀，否则部分框架解析会引起序列化错误。

```
e.g. Boolean isDeleted, it's method is isDeleted(), Error happened when RPC Framwork try to get property name deleted)
```

- 包名统一使用小写，点分隔符之间有且仅有一个自然语义的英语单词。包名统一使用单数形式，但是类名如果有复数含义，则类名可以使用复数形式。

```
e.g. com.alibaba.ai.util.MessageUtils
```

- 杜绝完全不规范的缩写，避免词不达义。

#### 【推荐】

- 为了达到代码自解释的目标，任何自定义编程元素在命名时，使用尽量完整的单词组合来表达其意。

```
e.g 在JDK中，对某个对象引用的volatile字段进行原子更新的类名为： AtomicReferenceFieldUpdater。
```

- 如果模块、接口、类、方法使用了设计模式 ，应在命名时体现出具体模式。

```
public class OrderFactory;
public class LoginProxy;
public class ResourceObserver;
```

- 接口类中的方法和属性不要加任何修饰符号(public 也不要加)， 保持代码的简洁性，并加上有效的 Javadoc 注释。尽量不要在接口里定义变量，如果一定要定义变量，必须是与接口方法相关的，并且是整个应用的基础常量。

```
接口方法签名：void commit();
接口基础常量：String COMPANY = "alibaba";
c.g. public abstract void commit();
如果JDK8中接口允许有默认实现，那么这个default方法，是对所有实现类都有价值的默认实现。
```

- 接口和实现类的命名有两套规则： - 【强制】对于 Service 和 DAO 类，基于 SOA 的理念，暴露出来的服务一定是接口，内部的实现类用以 Impl 后缀与接口区别。`e.g. CacheServiceImpl实现CacheService接口` - 【推荐】如果是形容能力的接口名称，取对应的形容词为接口名（通常是-able 的形式）。`e.g. AbstractTranslator 实现 Translatable`

#### 参考

- 枚举类名建议带上 Enum 后缀，枚举成员名称需要全大写， 单词间用下画线隔开。

```
枚举其实就是特殊的常量类，且构造方法被默认强制为私有。
e.g.枚举名字为ProcessStatusEnum的成员名称：SUCCESS / UNKNOWN_REASON
```

- 各层命名规约： - Service/DAO 层方法命名规约如下： - 获取单个对象的方法用 get 作为前缀。 - 获取多个对象的方法用 list 作为前缀。 - 获取统计值的方法用 count 作为前缀。 - 插入的方法用 sava / insert 作为前缀。 - 删除的方法用 remove / delete 作为前缀。 - 修改的方法用 update 作为前缀 - 领域模型命名规约如下: - 数据对象： xxxDO, xxx 为数据表名。 - 数据传输对象： xxxDTO, xxx 为业务领域相关的名称。 - 展示对象： xxxVO, xxx 一般为网页名称。 - `POJO` 是 DO/DTO/BO/VO 的统称，禁止命名成 xxx`POJO`。

### 1.2 常量定义

#### 强制

- 不允许任何魔法值（即未经预先定义的常量）直接出现在代码中。
- long 或者 Long 初始赋值时，使用大写的`L`，不能是小写的`l`。小写的`l`容易跟数字`1`混淆，造成误解。

#### 推荐

- 不要使用一个常量类维护所有常量，要按常量功能进行归类，分开维护。
- 常量的复用层次有 5 层：跨应用共享常量、应用内共享常量、子工程内共享常量、包内共享常量、类内共享常量。 - 跨应用共享常量：放置在二方库中，通常是在 client.jar 中的 constant 目录下。 - 应用内共享常量：旋转在一方库中，通常是在子模块中的 constant 目录下。 - 子工程内部共享常量：即在当前子工程的 constant 目录下。 - 包内共享常量：即在当前包下单独的 constant 目录下。 - 类内共享常量：直接在类内部以`private static final`的方式定义。
- 如果变量值仅在一个范围内变化，则用 enum 类型来定义。如果存在名称之外的延伸属性，则使用 enum 类型，下面正例的数字就是延伸信息，表示一年中的第几个季节。

```
public enum SeasonEnum {
    SPRINT(1), SUMMER(2), AUTUMN(3), WINTER(4);
    private int seq;
    SeasonEnum(int seq){
        this.seq = seq;
    }
}
```

### 1.3 代码格式

#### 强制

- 大括号的使用约定。如果大括号内为空，则简洁地写成{}即可，也不需要换行；如果是非空代码，则： - 左大括号前不换行。 - 左大括号后换行。 - 右大括号前换行。 - 右大括号后还有 else 等代码刚不换行；表示终止的右大括号必须换行。
- 左小括号和字符之间不出现空格；同样，右小括号和字符之间也不出现空格。
- if / for / while / switch / do 等保留字与括号之间都必须加上空格。
- 任何二目、三目运算符的左右两边都需要加一个空格。采用 4 个空格缩进，禁止使用 Tab 控制符。
- 注释的双斜线与注释内容之间有且仅有一个空格。

```
public static void main(String[] args){
    // 缩进4个空格
    String say = "hello";
	// 运算符的左右必须有1个空格
	int flag = 0;
	// 关键字 if 与括号之间必须有 1 个空格，
	// 括号内的 f 与左括号、0 与右括号之间不需要空格
	if (flag == 0) {
        System.out.println(say);
	}
	// 左大括号前加空格且不换行；左大括号后换行
	if (flag == 1) ｛
		System.out.println("world");
	// 右大括号前换行，右大括号后有else，不用换行
	} else {
        System.out.println("ok");
    // 在右大括号后直接结束，则必须换行
	}
}
```

- 单行字符数不超过 120 个，超出则需要换行，换行是遵循如下原则： - 第二行相对第一行缩进 4 个空格，从第三行开始，不再持续缩进，参考示例。 - 运算符与下文一起换行。 - 方法调用的点符号与下方一起换行。 - 方法调用中的多个参数需要换行时，在逗号后进行。 - 在括号前不要换行。

```
StringBuffer sb = new StringBuffer();
sb.append("you ").append("are ")...
    .append("so ")...
    .append("kind!");
```

- 方法参数在定义和传入时，多个参数逗号后面必须加空格。

```
method("one", "two", "three");
```

- IDE 的 text file encoding 设置为 UTF-8；IDE 中文件的换行符使用 UNIX 格式， 不要使用 Windows 格式。

#### 推荐

- 没有必要增加若干空格来使某一行的字符与上一行对应位置的字符对齐。
- 不同逻辑、不同语义、不同业务的代码之间插入一个空行分隔开来，以提升可读性。（没必要插入多个空行来隔开。）

### OOP 规约

#### 强制

- 避免通过一个类的对象引用访问此类的静态变量或静态方法，造成无谓增加编译器解析成本，直接用类名来访问即可。
- 所以的覆写方法，必须加`@Override`注解
- 相同参数类型，相同业务含义，才可以使用 Java 的可变参数，避免使用 Object。

```
可变参数必须放置在参数列表的最后（建议工程师们尽量不用可变参数编程）。
public User listUsers(String type, Long... ids){...}
```

- 对外部正在调用或者二方库依赖的接口，不允许修改方法接口，以避免对接口调用方产生影响。若接口过时，必须加 `@Deprecated`注解，并清晰地说明采用的新接口或者新服务是什么。
- 不能使用过时的类或方法。
- Object 的 equals 方法容易抛空指针，应使用常量或者确定有值的对象来调用 equals。

```
e.g. "test".equals(object)
c.g. object.equest("test");
```

- 所有相同类型的包装类对象之间值的比较，全部使用 equals 方法。

```
对于 Integer var = ? 在 - 128~127范围内的赋值， Integer 对象是在IntegerCache.cache中产生 的。会复用已有对象， 这个区间内的Integer值可以直接使用 == 进行判断， 但是这个区间外的所有数据， 都会在堆上产生， 并不会利用已有对象。这是个大坑，推荐使用equals方法进行判断。
```

- 关于基本数据类型与包装数据类型的使用标准如下： - 【强制】所有的`POJO`类属性必须使用包装数据类型。 - 【强制】RPC 方法的返回值和参数必须使用包装数据类型。 - 【推荐】所有的局部变量使用基本数据类型。
- 在定义 DO/DTO/VO 等 `POJO`类时，不要设定任何属性默认值。

```
c.g. `POJO`类的gmtCreate默认值为 new Date();但是这个属性在数据提取时并没有置入具体值，在更新其它字段时又附带更新了此字段，导致创建时间被修改成当前时间。
```

- 当序列化类新增属性时，请不要修改`serialVersionUID`字段，以避免反序列失败；如果完全不兼容升级， 避免反序列化混乱，那么请修改`serialVersionUID`值

```
注意`serialVersionUID`不一致会抛出序列化运行时异常。
```

- 构造方法里禁止加入任何业务逻辑，如果有初始化逻辑，请放在 init 方法中。
- `POJO`类必须写 `toString`方法。在使用 IDE 中的工具 `source> generate toString`时， 如果继承了另一个`POJO`类，注意在前面加一下 super.toString。

#### 推荐

- 当使用索引访问用 String 的 split 方法等到的数组时，需在最后一个分隔符后面做有无内容的检查，否则会有抛 `IndexOutofBoundsException`的风险。
- 当一个类有多个构造方法，或者多个同名方法时，这些方法应该按顺序放置在一起，便于阅读。
- 类内方法定义的顺序是： 公有方法或保护方法＞私有方法＞ getter / setter 方法。
- 在 setter 方法中， 参数名称与类成员变量名称一致，this.成员名 = 参数名。在 getter / setter 方法中，不要增加业务逻辑， 否则会增加排查问题的难度。
- 在循环体内，字符串的连接方式使用`StringBuilder`的`append`方法进行扩展。
- final 可以声明类、成员变量、方法及本地变量，下列情况使用 final 关键字： - 不允许被继承的类，如：`String`类。 - 不允许修改引用的域对象， 如：`POJO`类的域变量。 - 不允许被重写的方法，如： `POJO`类的 setter 方法。 - 不允许运行过程中重新赋值的局部变量。 - 避免上下文重复使用一个变量，使用 final 描述可以强制重新定义一个新变量，方便更好地进行重构
- 慎用 Object 的 clone 方法来拷贝对象。
- 类成员与方法访问控制从严： - 如果不外部直接通过 new 来创建对象，那么构造方法必须限制为`private`。 - 工具类不允许有 public 或 default 构造方法。 - 类非`static`成员变量并且与子类共享，必须限制为`protected`。 - 类非`static`成员变量并且仅在本类使用，必须限制为`private`。 - 类`static`成员变量如果公在本类使用，必须限制为`private`。 - 若是`static` 成员变量，必须考虑是否为`final`。 - 类成员方法只供类内部调用，必须限制为`private`。 - 类成员方法只对继承类公开，限制为`protected`.

### 1.5 集合处理

#### 强制

- 关于`hashCode` 和`equals`的处理，遵循如下规则： - 只要重写 equals, 就必须重写 hashCode。 - 因为 `Set`存储的是不重复的对象，依据 hashCode 和 equals 进行判断，所以 Set 存储的对象必须重写这两种方法。 - 如果自定义对象作为`Map`的键，那么必须重写 hashCode 和 equals。
- `ArrayList`的`subList`结果不可强转成`ArrayList`, 否则会抛出`ClassCastExcpetion`异常，即

```
java.util.RandomAccessSubList cannot be cast to java.util.ArrayList`
```

- 在`subList`场景中，高度注意对原集合元素个数的修改，会导致子列表的遍历、增加、删除均产生`ConcurrentModificationException`异常。
- 使用集合转数组的方法，必须使用集合的`toArray(T[] array)`，传入的是类型完全一样的数据，大小就是 list.size()。
- 在使用工具类`Arrays.asList()`把数组转换成集合时，不能使用其修改集合相关的方法，它的`add`/`remove`/`clear`方法会抛出`UnsupportedOperationException`异常。

```
asList 的返回对象是一个Arrays内部类，并没有实现集合的修改方法。Arrays.asList体现的是适配器模式，只是转换接口，后台的数据仍是数组。
String[] strs = new String[] {"you", "are"};
List list = Arrays.asList(strs);
scenario 1: list.add("chenxiong") 运行时异常。
scenario 2: 如果 str[0] = "xiche"; 那么list.get(0)也会随之修改。
```

- 泛型通配符`<? extends T>`用来接收返回的数据，此写法的泛型集合不能使用 add 方法，而`<? super T>`不能使用 get 方法， 因为 其作为接口调用赋值时易出错。

```
扩展说一下 PECS(Producer Extends Consumer Super)原则：
	第一，频繁往外读取内容的，适合用<? extends T>;
	第二，经常往里插入的，适合用<? super T>。
```

- 不要在`foreach`循环里进行元素的`remove`/`add`操作。remove 元素请用 `Iterator`方式，如果并发操作，需要对 Iterator 对象加锁。
- 在 JDK7 及以上版本中， `Comparator`要满足如下三个条件，不然`Arrays.sort`, `Collections.sort`会报`IllegalArgumentException`异常。

```
三个条件如下：
	1) x, y 的比较结果和y, x的比较结果相反。
	2) x>y, y>z, 则 x>z。
	3) x=y, 则x, z 比较结果和y, z比较结果相同
c.g. 下例中没有处理相等的情况，交换两个对象判断结果并不互反，不符合第一个条件，在实际使用中可能会出现异常。
	new Comparator<Student>(){
        @Override
        public int compare(Student o1, Student o2){
            return 01.getId() > o2.getId() ? 1:-1;
        }
	}
```

#### 推荐

- 在集合初始化时，指定集合初始值大小。
- 使用`entrySet`遍历`Map`类集合 K/V， 而不是用 keySet 方式遍历。

```
keySet 其实遍历了2次，一次是转为Iterator对象，另一次是从hashMap中取出key反对应的value。如果是JDK8，使用Map.foreach方法。
```

- 高度注意`Map`类集合 K/V 能不能存储 null 值的情况

  | 集合类            | Key     | Value   | Super       | Comment               |
  | ----------------- | ------- | ------- | ----------- | --------------------- |
  | Hashtable         | no null | no null | Dictionary  | Thread Safe           |
  | ConcurrentHashMap | no null | no null | AbstractMap | 锁分段技术 (JDK8:CAS) |
  | TreeMap           | no null | null    | AbstractMap | Thread Unsafe         |
  | HashMap           | null    | null    | AbstractMap | Thread Unsafe         |

```
由于HashMap的干扰，很多人认为ConcurrentHashMap是可以置入null值的，而事实上，存储null值会抛出NPE异常
```

#### 参考

- 合理利用集合的有序性(sort)和稳定性(order)，避免集合的无序性(unsort)和不稳定性(unorder)带来的负面影响。

```
有序性批遍历的结果是按某种比较规则依次排列的。稳定性指的是集合每次遍历的元素次序是一定的。如：
ArrayList 是 order/unsort;
HashMap 是unorder/unsort;
TreeSet 是order/sort.
```

- 利用 Set 元素唯一的特性，可以快速对一个集合进行去重操作，避免使用 List 的 contains 方法进行遍历、对比、去重操作。

未完待续...└(^o^)┘
