---
layout: post
title:  "重构大法练级1"
date:   201-06-12 19:30 +0800
categories: 学无止境
tag: 周边
---

* content
{:toc}


人们常说，善于总结的人更优秀..
把所有做的事情比做一个西瓜的话，西瓜切成十块，我吃下去的三块大概就是我觉得有必要记录总结一下的事情，而真的付诸行动的就像我意外吃下去的西瓜籽那么多.. 现在无籽西瓜很普遍了，看来吃西瓜籽的机会都要没了。
可能这个比喻不太合适，如果我说以后要多总结，那不是告诉自己要多吃西瓜籽。


废话太多了，下面开始正文。
周末看了下《重构》，就想着实践下，于是就实践了下。回看代码时，试着对比“代码里的坏味道”，找出几个容易发现并卓有成效的几个点：1.过长函数 2.过长参数列 3.数据泥团 4.过多的注释 5.简化条件表达式 6.重复代码


#### 过长函数
> 你应该更积极进取地分解函数。我们遵循这样一条原则：每当感觉需要以注释来说明点什么的时候，我们就把需要说明的东西写进一个独立函数中，并以其用途（而非实现手法）命名。我们可以对一组或甚至短短一行代码做这件事。哪怕替换后的函数调用动作比函数自身还长，只要函数名称能够解释其用途，我们也该毫不犹豫地那么做。关键不在于函数的长度，而在于函数「做什么」和「如何做」之间的语义距离。

看个例子：
重构前的一个方法里，有一段设置值的代码，由于要设置的配置值比较多，看起来会比较长且凌乱，真的有点Nausea

```
template.set(String.format(ConfigurationKeyConstants.DATAX_X_PARAMETER_DEFAULTFS, getModeByType(type)),getDefaultFs(frontConfigs));

template.set(String.format(ConfigurationKeyConstants.DATAX_X_PARAMETER_FILETYPE, getModeByType(type)),
        frontConfigs.getString(ConfigurationKeyConstants.FRONT_HIVE_FILETYPE,
                type instanceof DsType.DsSourceType ? DEFAULT_READER_FILE_TYPE : config.getDefaultHdfsFileType()));

template.set(String.format(ConfigurationKeyConstants.DATAX_X_PARAMETER_FILENAME, getModeByType(type)),config.getDefaultHdfsFileWriteName());
template.set(String.format(ConfigurationKeyConstants.DATAX_X_PARAMETER_FIELDDELIMITER, getModeByType(type)),
        frontConfigs.getString(ConfigurationKeyConstants.FRONT_HIVE_FIELDSDELIMITER, DEFAULT_FILE_DELIMITER));

template.set(String.format(ConfigurationKeyConstants.DATAX_X_PARAMETER_SKIPHEADER, getModeByType(type)),
        frontConfigs.getBool(ConfigurationKeyConstants.FRONT_TXT_CONTAIN_HEADERS, false));
template.set(String.format(ConfigurationKeyConstants.DATAX_X_PARAMETER_HADOOP_CONFIG, getModeByType(type)), getHadoopConfig(frontConfigs));

if (type instanceof DsType.DsTargetType) {
    //临时目录
    template.set(ConfigurationKeyConstants.DATAX_W_PARAMETER_TMPDIR_CONFIG, config.getHdfsTmpDir());
    //写入模式
    template.set(String.format(ConfigurationKeyConstants.DATAX_X_PARAMETER_WRITEMODE, getModeByType(type)),
            getWriteModeName(frontConfigs));
    //拼接分区的path
    template.set(String.format(ConfigurationKeyConstants.DATAX_X_PARAMETER_PATH, getModeByType(type)),
            basePath + buildPathByPartitionFields(allConfigs));
}
if (type instanceof DsType.DsSourceType) {
    template.set(String.format(ConfigurationKeyConstants.DATAX_X_PARAMETER_PATH, getModeByType(type)), basePath);
}
...
```
重构后，将取值都替换成一个个小的方法，一下舒服多了
```
template.set(defaultFsKey(type), defaultFsVal(frontConfigs));
template.set(fileTypeKey(type), fileTypeVal(type, frontConfigs));
template.set(fileNameKey(type), config.getDefaultHdfsFileWriteName());
template.set(fieldDelimiterKey(type), fieldDelimiterVal(frontConfigs));
template.set(skipHeaderKey(type), skipHeaderVal(frontConfigs));
template.set(hadoopConfigKey(type), getHadoopConfig(frontConfigs));

writerOnlyConfig(processData);
readerOnlyConfig(processData);
```
同时对重复的String.format代码提取了一个一眼就知道干了啥的公共方法去替换，例如
```
private String skipHeaderKey(DsType type) {
    return formatConfigKeyByPluginType(type, ConfigurationKeyConstants.DATAX_X_PARAMETER_SKIPHEADER);
}
protected String formatConfigKeyByPluginType(DsType pluginType, String key) {
    return String.format(key, pluginType.pluginPrefixName());
}
``` 


#### 过长参数列&数据泥团
> 过长参数列：太长的参数列难以理解，太多参数会造成前后不一致、不易使用，而且一旦你需要更多数据，就不得不修改它。如果将对象传递给函数，大多数修改都将没有必要，因为你很可能只需（在函数内）增加一两条请求（requests），就能得到更多数据。
数据泥团：数据项（data items）就像小孩子：喜欢成群结队地待在一块儿。你常常可以在很多地方看到相同的三或四笔数据项：两个classes内的相同值域（field）、许多函数签名式（signature）中的相同参数。这些「总是绑在一起出现的数据」真应该放进属于它们自己的对象中。一个好的评断办法是：删掉众多数据中的一笔。其他数据有没有因而失去意义？如果它们不再有意义，这就是个明确信号：你应该为它们产生一个新对象。


很不幸，在我对代码重构时，发现过长参数列的坏味道同时就是数据泥团..
重构前，代码里有好几个地方是这样的参数泥团
```
void convert(JobConfigurationContainer template, JobConfigurationContainer allConfigs, DsType readerType, DsType writerType) {...}
void convertPluginConfiguration(JobConfigurationContainer template, JobConfigurationContainer allConfigs, DsType type) {...}
```
将这组参数列都放入一个对象中后,世界又美好了一点.(原谅那略显别扭的命名..)
```
void convertPluginConfiguration(JobConfigurationProcessData processData) {...}
```


#### 简化条件表达式
>「复杂的条件逻辑」是最常导致复杂度上升的地点之一。你必须编写代码来检查不同的条件分支、根据不同的分支做不同的事，然后，你很快就会得到一个相当长的函数。大型函数自身就会使代码的可读性下降，而条件逻辑则会使代码更难阅读。在带有复杂条件逻辑的函数中，代码（包括检查条件分支的代码和真正实现功能的代码）会告诉你发生的事，但常常让你弄不清楚为什么会发生这样的事, 这就说明代码的可读性的确大大降低了。
和任何大块头代码一样，你可以将它分解为多个独立函数，根据每个小块代码的用 途，为分解而得的新函数命名，并将原函数中对应的代码替换成「对新建函数的调用」，从而更清楚地表达自己的意图
重构前，我有一个这样的表达式，并不能一眼看出这是做什么：
```
if (null != frontColumn.getString(ConfigurationKeyConstants.FRONT_FIELD_ISPARTITION)
        && frontColumn.getBool(ConfigurationKeyConstants.FRONT_FIELD_ISPARTITION))
```
简化后：
```
if (existPartitionField(frontColumn) && isPartition(frontColumn)) 
```

#### 总结
这次重构只对几个简单且易于发现的“坏味道”进行了重构，其他一些高级的“坏味道”还没有很好的理解透，等再修炼一番再战

