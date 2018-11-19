---
layout: post
title: Design Patterns(Proxy)
subtitle: Static Proxy and Dymatic Proxy in python
date: 2018-11-19
author: BF
header-img: img/bf/road_01.jpg
catalog: true
tags:
  - design patterns
  - proxy
  - java
  - python
---

# 前言

早些年看《大话设计模式》的时候，用`Java`实现了书中的例子，可惜当年没有保存好，好多东西都丢了。

现在想着用`python`重新实现一下 ，重新学习。

虽然 python 没有接口的概念，但是可以用抽象类代替。

# 代理模式

代理模式是给某一个对象提供一个代理对象，并由代理对象控制对原对象的引用。通俗的来讲代理模式就是我们生活中常见的中介。

![proxy](img/post/2018/11/2018-11-19-DesignPattern-Proxy.jpg)

**为什么要用代理模式？**

**中介隔离作用：**

在某些情况下，一个客户类不想或者不能直接引用一个委托对象，而代理类对象可以在客户类和委托对象之间起到中介的作用，其特征是代理类和委托类实现相同的接口。

**开闭原则，增加功能：**

代理类除了是客户类和委托类的中介之外，我们还可以通过给代理类增加额外的功能来扩展委托类的功能，这样做我们只需要修改代理类而不需要再修改委托类，符合代码设计的开闭原则。
代理类主要负责为委托类预处理消息、过滤消息、把消息转发给委托类，以及事后对返回结果的处理等。代理类本身并不真正实现服务，而是同过调用委托类的相关方法，来提供特定的服务。
真正的业务功能还是由委托类来实现，但是可以在业务功能执行的前后加入一些公共的服务。例如我们想给项目加入缓存、日志这些功能，我们就可以使用代理类来完成，而没必要打开已经封装好的委托类。

**有哪几种代理模式？**

我们有多种不同的方式来实现代理。如果按照代理创建的时期来进行分类的话， 可以分为两种：**静态代理**、**动态代理**。

# 静态代理

静态代理是由程序员创建或特定工具自动生成源代码，在对其编译。

下面这个是以购买房子为例子：

```python
from abc import ABCMeta,abstractmethod

class IBuyHouseMan(metaclass=ABCMeta):
    @abstractmethod
    def pay_money(self, money):
        '''
        :param money:
        :return:
        '''
        raise NotImplementedError

class BuyHouseManImpl(IBuyHouseMan):

    def pay_money(self, money):
        print("pay money({}) for house!".format(money))

class BuyHouseManProxy(IBuyHouseMan):

    def __init__(self, buy_house_man):
        self.buy_house_man = buy_house_man

    def pay_money(self, money):
        print("prepare contract")
        self.buy_house_man.pay_money(money)
        print("get commission ")

def test():
    buy_house_man = BuyHouseManImpl()
    buy_house_man.pay_money(1000)

    buy_house_proxy = BuyHouseManProxy(buy_house_man)
    buy_house_proxy.pay_money(1000)

if __name__ == "__main__":
    test()
```

# 动态代理

动态代理是在程序运行时通过反射机制动态创建的，耦合性大大降低，更加灵活。

我对 python 的反射还不太熟悉，下面是网上的例子，有时候重构下，感觉他写的有点麻烦和绕：

```python
from abc import ABCMeta,abstractmethod
import types

class InvocationHandler:
    def __init__(self, obj, func):
        self.obj = obj
        self.func = func
    def __call__(self, *args, **kwargs):
        print('handler:', self.func, args, kwargs)
        print('before call the func.')
        self.func(*args, **kwargs)
        print('after call the func.')

class HandlerException(Exception):
    def __init__(self, cls):
        super(HandlerException, self).__init__(cls,  'is not a hanlder class')

class Proxy:
    def __init__(self, cls, hcls):
        self.cls = cls
        self.hcls = hcls
        self.handlers = dict()

    def __call__(self, *args, **kwargs):
        self.obj = self.cls(*args, **kwargs)
        return self

    def __getattr__(self, attr):
        print('get attr:', attr)
        isExist = hasattr(self.obj, attr)
        res = None
        if isExist:
            res = getattr(self.obj, attr)
            if isinstance(res, types.MethodType):
                if self.handlers.get(res) is None:
                    self.handlers[res] = self.hcls(self.obj, res)
                return self.handlers[res]
            else:
                return res
        return res

class ProxyFactory:
    def __init__(self, hcls):
        if issubclass(hcls, InvocationHandler) or hcls is InvocationHandler:
            self.hcls = hcls
        else:
            raise HandlerException(hcls)

    def __call__(self, cls):
        return Proxy(cls, self.hcls)


@ProxyFactory(InvocationHandler)
class BuyHouseMan:
    def __init__(self, money):
        self.money = money
    def pay_money(self):
        print("pay money({}) for house!".format(self.money))


def test():
    buy_house_man  = BuyHouseMan(1000)
    print(type(buy_house_man))
    buy_house_man.pay_money()

if __name__ == "__main__":
    test()
```

如果使用 Java 的话，那么就更加简单一些，应该类库中就有现成现成的`InvocationHandler`，很方便：

```java
package main.java.proxy.impl;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;

public class DynamicProxyHandler implements InvocationHandler {

    private Object object;

    public DynamicProxyHandler(final Object object) {
        this.object = object;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("before invoke");
        Object result = method.invoke(object, args);
        System.out.println("after invoke");
        return result;
    }
}
```

测试类：

```java
package main.java.proxy.test;

import main.java.proxy.BuyHouse;
import main.java.proxy.impl.BuyHouseImpl;
import main.java.proxy.impl.DynamicProxyHandler;

import java.lang.reflect.Proxy;

public class DynamicProxyTest {
    public static void main(String[] args) {
        BuyHouse buyHouse = new BuyHouseImpl();
        BuyHouse proxyBuyHouse = (BuyHouse) Proxy.newProxyInstance(BuyHouse.class.getClassLoader(), new
                Class[]{BuyHouse.class}, new DynamicProxyHandler(buyHouse));
        proxyBuyHouse.buyHosue();
    }
}
```

# CGLIB代理

待填坑。

**参考:**

- [设计模式---代理模式](https://www.cnblogs.com/daniels/p/8242592.html)

- [Python：动态代理机制](https://blog.csdn.net/water_likud/article/details/80566177)
