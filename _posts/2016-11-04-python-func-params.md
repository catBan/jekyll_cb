---
layout: post
title:  "自定义函数参数"
date:   2016-11-04 +0800
categories: 学无止境
tag: Python
---

* content
{:toc}


﻿

### #1.默认参数

----计算x的n次方

```

def power(x,n=2):

	s= 1

	while n>0:

		n= n-1

		s = s*x1

	return s

```

默认参数的使用

```

def enroll(name,gender,age=6,city="beijing"):

	print('name:',name)

	print('gender:',gender)

	print('age:',age)

	print('city:',city)

```

两种调用方式

```

enroll("Jie","M",7)

enroll("Ke","M",city="tianjin")

```

### # 2.可变参数

----*parameter 

```

def calc(*numbers):

	sum = 0

	for n in numbers:

		sum = sum +n*n

	return sum

calc(1,3,5)

calc(3,5)

```

python可在list或tuple前加'*',直接转换成可变参数

```

nums = [1,2,4]

calc(*nums)

```

### # 3.关键字参数

---- **parameter



调用时 **parameter自动封装成tuple

```

def person(name,age,**kw):

	print('name:',name,'age',age,'other',kw)



person('Li',21,birth='1992',city = 'shanghai')



dict1 = {'city':'shanghai','birth':'1992'}

person('Li',21,birth=dict1['birth'],city = dict1['city'])

person('Li',21,**dict1)

```

\*\*dict1表示把dict1这个dict的所有key-value用关键字参数传入到函数的**kw参数，kw将获得一个dict，

> 注意kw获得的dict是dict1的一份拷贝，对kw的改动不会影响到函数外的dict1。



### # 4.命名关键字参数

----限制传入的关键字参数，只允许传入指定的；命名关键字参数必须传入参数名

```

def person(name,age,*,city,job):

	print(name,age,city,job)  

```

用'*'分隔命名关键字参数。

```

person('Jack',24,city='Xiamen',job='Engineer')

```

如果定义时，命名关键字参数没有默认值，则需要全部传入。反之可不传默认参数

```

def person(name,age,*,city="beijing",job):

	print(name,age,city,job)  

person('Da',24,job='Engineer')

```

如果函数定义中已经有一个可变参数，后面定义的命名关键字参数不需要再加特殊分隔号‘*’

```

def person(name,age,*args,city,job):

	print(name,age,args,city,job)

```

### # 5.参数组合

----参数定义的顺序必须是：必选参数、默认参数、可变参数、命名关键字参数和关键字参数。

```

def f1(a,b,c=0,*args,**kw):

	print('a=',a,'b=',b,'c=',c,'args=',args,'kw=',kw)

def f2(a,b,c=0,*,d,**kw):

	print('a =', a, 'b =', b, 'c =', c, 'd =', d, 'kw =', kw)

f1(1,2)

f1(1, 2, c=3)

f1(1, 2, 3, 'a', 'b')

f1(1, 2, 3, 'a', 'b', x=99)

f2(1, 2, d=99, ext=None)

```

还可以通过tuple,dict调用

```

args = (1, 2, 3, 4)

kw = {'d': 99, 'x': '#'}

f1(*args, **kw)



args = (1, 2, 3)

kw = {'d': 88, 'x': '#'}

f2(*args, **kw)

```

所以，对于任意函数，都可以通过类似func(*args, **kw)的形式调用它，无论它的参数是如何定义的。



