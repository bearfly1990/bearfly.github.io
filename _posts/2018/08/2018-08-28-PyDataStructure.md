---
layout: post
title: Data Structure In Python
subtitle: List/Dictionaries(Map)/Set/Queue/Stack... in python
date: 2018-08-28
author: BF
header-img: img/bf/beach_03.jpg
catalog: true
tags:
  - python
  - data structure
---

# Data Structure In Python

Today I met a problem that I would like to count the duplicate items in a list. Generally, I could iterate the list and count it by myself.

But I don't think that's a good idea, and python perhaps have the easiest way to do this.

So I searched for the solution and find it. there is a build-in function to do this.

```python
>>> list_test = ['test','test','xiche','xiche','xiche',1,1,1,1,1,['a']]
>>> print(list_test.count('test'))
2
>>> print(list_test.count('xiche'))
3
>>> print(list_test.count(1))
5
>>> print(list_test.count(['a']))
1
```

So it's important to be familar with the usage of data structure in python and it will help you to code more efficient.

And let me summarize some basic operations about them and I will continue to update this article when I find some new interesting tips.

Learn to look into offical document is very import.

## List

`List` is a basic data structure, and in python, it's very flexible and could use it as below:

```python
def list_test():
    list_01 = []

    list_01.append('a')
    list_01.append('b')
    list_01.remove('b')
    list_02 = [1]
    list_03 = list_01 + list_02
    print(list_03)
    #['a', 1]
    for item in list_03:
        print(item)
    #a
    #1
    print(len(list_03))
    #2
    list_04 = list_03.copy() # list_04 = list_03[:]
    list_03.clear()
    print(list_03)
    #[]
    print(list_04)
    #['a', 1]
    list_04.reverse()
    print(list_04)
    #[1, 'a']

    #list_04.sort()
    #print(list_04)
    #Traceback (most recent call last):
    #  File ".\python_structure.py", line 25, in <module>
    #    list_04.sort()
    #TypeError: '<' not supported between instances of 'str' and 'int'

    list_04 = [2,3,4,9]
    list_04.sort()
    print(list_04)
    #[2, 3, 4, 9]
    list_04.sort(reverse=True)
    print(list_04)
    #[9, 4, 3, 2]

    print(list_04.index(3))
    #2
```

## Stack

As we know, Stack is always based on `List` in most languages and the features is:

`LIFO: Last In First Out`

we could use method in `List` to simulate easily by only use `append()` and `pop()`

```python
def stack_test():
    stack = []
    stack.append(1)
    stack.append(2)
    stack.append(3)
    print(stack)
    #[1, 2, 3]
    value = stack.pop()
    print(value)
    #3
    print(stack)
    #[1, 2]
    stack.append(4)
    value = stack.pop()
    print(value)
    #4
    print(stack)
    #[1, 2]
```

## Queue

Queue is another basic data structure, there is a module `queue` to handle this:

`FIFO: First In First Out`

```python
import queue
def queue_test():
    queue_01 = queue.Queue()
    queue_01.put(1)
    queue_01.put(2)
    queue_01.put(3)
    value = queue_01.get()
    print(value)
    #1
    queue_01.put(4)
    value = queue_01.get()
    print(value)
    print(queue_01.empty())
    #False
```

## Dictionaries(Map)

Dictionaries is another import data structure in python.

```python
def dictionary_test():
    fruits = {'banana': 1, 'apple': 2}
    print(fruits)
    #{'banana': 1, 'apple': 2}
    print(fruits['apple'])
    #1
    print(list(fruits))
    #['banana', 'apple']
    print(sorted(fruits))
    #['apple', 'banana']
    del fruits['apple']
    print(fruits)
    #{'banana': 1}

    fruits = dict([('apple', 10), ('banana', 20), ('watermelon', 30)])
    print(fruits)
    #{'apple': 10, 'banana': 20, 'watermelon': 30}
    fruits = dict(apple=11, banana=21, watermelon=31)
    print(fruits)
    #{'apple': 11, 'banana': 21, 'watermelon': 31}

    for fruit_name in fruits:
        print(fruits[fruit_name])
    #11
    #21
    #31
```

## Sets

Sets is usually used to remove the duplicate values

```python
def sets_test():
    fruits = {'apple', 'orange', 'apple', 'pear', 'orange', 'banana'}
    print(fruits)
    #{'banana', 'orange', 'pear', 'apple'}
    print('apple' in fruits)
    #True

    fruits = ['apple', 'orange', 'apple', 'pear', 'orange', 'banana']
    print(fruits)
    #['apple', 'orange', 'apple', 'pear', 'orange', 'banana']
    fruits = set(fruits)
    print(fruits)
    #{'apple', 'orange', 'banana', 'pear'}

    sets_01 = {'a','b','c'}
    sets_02 = {'c','d','e'}

    print(sets_01 & sets_02)
    #'c'
    print(sets_01 | sets_02)
    #{'e', 'a', 'c', 'b', 'd'}
    print(sets_01 - sets_02)
    #{'b', 'a'}

```

## Tuples and Sequences

Tuples and sequences is also flexible.

```python
def test_tuples_sequences():
    tuples_01 = 100,200,'test'
    print(tuples_01)
    #(100, 200, 'test')
    print(tuples_01[2])
    #test
    tuples_02 = 300,400
    tuples_01 = tuples_01, tuples_02
    print(tuples_01)
    #((100, 200, 'test'), (300, 400))

    var_01,var_02,var_03 = 100,200,'test'
    print(var_01, var_02, var_03)
    #100 200 test
```

## To Be Continued...

please refer to [https://docs.python.org/3/tutorial/datastructures.html](https://docs.python.org/3/tutorial/datastructures.html) for more information.
