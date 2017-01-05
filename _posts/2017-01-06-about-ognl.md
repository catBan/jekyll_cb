---
layout: post
title:  "About OGNL"
date:   2017-01-07 00:18 +0800
categories: others
tag: 周边
---

* content
{:toc}


>今天看SpringMvc框架对于多对象数据绑定的拓展时，看到这样一个问题：Struct框架由于使用的是OGNL,通过跟对象操作，不存在无法多对象绑定的问题，而Spring MVC不是。那么问题来了，What is OGNL?

### OGNL

 Object-Graph Navigation Language. 

嗯 ，这是一种基于java对象属性get和set对表达式语言，外加其他如列表映射，lambda表达式。

Ognl类包含用于评估OGNL表达式的方便方法。 你可以在两个阶段中执行此操作，将表达式解析为内部形式，然后使用该内部形式设置或获取属性的值; 或者可以在单个阶段中执行，并使用表达式的String形式直接获取或设置属性。

### what OGNL is good for?

1.用于模型对象的GUI元素（文本字段，组合框等）之间的绑定语言。 OGNL的TypeConverter机制使得值从一种类型转换为另一种类型（例如，字符串转换为数字类型）变得更简单;

2.用于在表列和Swing TableModel之间映射的数据源语言;

3.Web组件和底层模型对象之间的绑定语言;

4.对Apache Commons BeanUtils包或JSTL的EL（仅允许简单的属性导航和基本的索引属性）使用的属性获取语言进行更具表达式的替换。



-------

### for example

以上是Apache Commons上对OGNL对介绍的简单翻译，看完之后细细研究一下，通俗的说OGNL，它是什么呢？大致就是让数据在对象跟表单元素或对象与Web组建等场景下的表达、绑定更简洁。

要怎么用？举几个我拿来试手等例子吧～

#### 1.先了解下OGNL基于上下文进行这件事～

OGNL的context其实就是Map对象，缺省时有`#root`跟`#context`;既然是个map对象，你可以往里添加你自己的上下文参数，以便在OGNL表达式中使用。

比如：

```

Map context = new HashMap();
context.put("bookContext","Catban's book store");
try {
   Book root = new Book();
   root.setId(1);
   root.setName("Java Book");

   //－返回 Catban's book store
   String myCtx = (String)Ognl.getValue("#bookContext", context,root);
   String myCtx2 = (String)Ognl.getValue("#context.bookContext", context,root);
   //－－－也可对表达式进行预解析，使其缓存起来，用起来更高效
   Object expression = Ognl.parseExpression("#bookContext");
   String myCtx3 = (String)Ognl.getValue(expression,context,root);
   System.out.println("1:"+myCtx3);

   //－返回Java Book
   String bookName = (String)Ognl.getValue("name", root);
   String bookName2 = (String)Ognl.getValue("#root.name", root);
   System.out.println("2:"+bookName2);

```



#### 2.更复杂的读取

比如读List。假设要给书分类，现在有个`Category`的类，里面有`List<Book>`的属性，现在我要取计算机分类下的所有书名：

```

Category category = new Category();
category.setBooks(computerBookList);

List names = (List)Ognl.getValue("Books.{name}", category);

```

你甚至可以写更具体的条件，比如删选书籍id>n的啦，这些具体用法自己再查吧。

#### 3.像Jstl一样在Jsp中使用OGNL

```


<s:set name="book" value="#{'book1':'java', 'book2':'py'}" />    

<s:property value="#book['book1']" />

//访问静态属性／方法

<s:property value="@Java.lang.Math@floor(4.55)"/>

<s:property value="@com.catban.Book@JAVA"/>

//当然也有判断啦，遍历之类的，具体就不细说了

```

#### 4.补充说明`#`



访问非根对象属性时，需要加`#`;

过滤和投影集合时：如`books.{?#this.id>3}`;

构造map时，如3中例子。


