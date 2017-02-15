---
layout: post
title:  "RequireJs 使用笔记"
date:   2017-02-15 20:52 +0800
categories: 学无止境
tag: 周边
---

* content
{:toc}


###  #关于data-main

`data-main`是`requireJs`的一个入口函数，这个参数对应的js是异步加载。

如果使用该参数来定义`entry ponit` ,存在一个问题：不能保证该js的加载顺序在`require.js`加载完成之后才加载；若它迟于`require.js`,有可能会出现找不到引用的脚本的问题。比如,

文档结构：

- js/

    - requirejs/

        -  **require.js**

        - **main.js**

        - **jquery-private.js**

    - **jquery-2.2.3.min.js**



入口点：

```

<script data-main="/js/requirejs/main" src="/js/require-2.3.2.js"></script>

```

main.js

```
requirejs.config({
   baseUrl: '/js',
   paths: {
       jquery : 'jquery-2.2.3.min',
       'jquery-private':'requirejs/jquery-private'
   },

   map:{
       '*':{'jquery':'jquery-private'},
        'jquery-private':{'jquery':'jquery'}
   }
});

```



demo.js

```
require(['jquery'], function( $ ) {
   //do sth.
});

```

结果：有可能出现查找jquery 404,此时`require.js`以`main.js`的所在目录查找`jquery`

```

GET http://localhost:8080/js/plugins/requirejs/jquery.js 404 (Not Found)

```

如果文件只有一个入口脚本，可以使用`data-main` ,如果不是，最好不要用这个属性；那么要用什么呢？下面会说。



###  # 两个方式配置require.js

#### 1.先加载require.js,通过`requirejs.config`配置

```

<script src="/js/requirejs/require-2.3.2.js"></script>
<script src="/js/requirejs/main.js"></script>

```

`main.js` : 同前面的`main.js`配置

#### 2.后加载require.js,通过变量`require`配置

 

```

<script src="/js/requirejs/main.js"></script>

<script src="/js/requirejs/require-2.3.2.js"></script>

```

`main.js` :定义变量

```
var require = {
  baseUrl: '/js',
   paths: {
       jquery : 'jquery-2.2.3.min',
       'jquery-private':'requirejs/jquery-private'
   },

   map:{
       '*':{'jquery':'jquery-private'},
        'jquery-private':{'jquery':'jquery'}
   }

}

```



以上两种配置可以避免异步加载main.js带来的问题



> **Tips** 在jquery与require.js合成的时候，不要把jquery脚本包在目录`jquery`里面，包在`jqueryXX` 什么的都可以



### # 使用define时出现Uncaught Error: Mismatched anonymous define() module

这个错误把define改成require时不会有。。。纠结了好久，文档里的[常见错误](http://requirejs.cn/docs/errors.html#mismatch)  也看了，但都不是描述的问题。

 *这个问题后来解决了，不是典型的出错方式，是由于没把`require`/ `define`的使用场景分清楚，导致的一些误解*



### #引入未模块化的第三方依赖 - shim

使用`shim`可以包装未按`requireJs`要求`define()`的脚本，但是在使用时，包装了还是会报一个找不到`jQuery`的错误  - -

```
requirejs.config({
   baseUrl: '/js',
   paths: {
       //jquery must define to 'jquery',or it will appear 'undefined' error.Because jquery自身就是有主模块
       jquery : 'plugins/jquery-2.2.3.min',
       'jquery-private':'plugins/requirejs/jquery-private',
       Class : 'view/ClassTool',
       jqValidate : 'plugins/jquery_validate/jquery.validate.min',
       validateExt : 'plugins/jquery_validate/jquery.validate.extend',
       bootstrap : 'plugins/bootstrap/js/bootstrap'
   },
   shim:{
       'bootstrap':{
           deps:['jquery']
       }
   },
   // jquery-private 清除全局变量 $ 和 jQuery
   // 除了jquery-private之外的任何依赖中，还可以直接使用 jquery 这个模块名，并且总是被替换为对 jquery-private 的依赖，使得它最先被执行。
   map:{
       '*':{'jquery':'jquery-private'},
       'jquery-private':{'jquery':'jquery'}
   }
});

```

仔细查验配置，跟官方文档的一致，觉得问题不是出在配置上；怀疑是`jquery-private`这个清楚全局变量的配置影响了。

`jquery-private`只做了一件事，清除`$`/`jQuery`:

```
define(['jquery'], function(jQuery) {
   return jQuery.noConflict(true);
});

```

去掉`jquery-private`后，问题果然解决了。但是，这两个配置我都要呢？

先来看下`jQuery`的`noConflict`都做了什么：

```
var

   // Map over jQuery in case of overwrite
   _jQuery = window.jQuery,

   // Map over the $ in case of overwrite
   _$ = window.$;

jQuery.noConflict = function( deep ) {
   if ( window.$ === jQuery ) {
       window.$ = _$;
   }

   if ( deep && window.jQuery === jQuery ) {
       window.jQuery = _jQuery;
   }

   return jQuery;
};

```

执行`noConflict(true)`后，`window.$`以及`window.jQuery`被清除掉了，此时当前`jQuery`对`$`跟`jQuery`的控制权都将失效，转移给第一个产生它们的库。而它返回了原始的`jQuery`,此时如果定义一个变量接收它，便可用此变量操作：

```

var jq = jQuery.noConflict(true);

jq("#mainFrame");

//...

```

而我的项目中不存在其它库产生了`$`或`jQuery`,所以在引入了`jquery-private`后会产生`jQuery`undefine的异常。针对遇到的情况，暂时可行的解决方案想到几个：

1.在没有冲突的前提下，不使用`jquery-private`来做清除

2.保留`jquery-private`，在需要`jquery`的非规范依赖上，加上一层`define()`包装：

```

define(['jquery'],function(jQuery){

//....

})

```

3.将`jquery-private`中对 `$`、jQuery的清除，改为仅对$清除,同时需要对非规范依赖进行`define()`,导出`$`：

```

return jQuery.noConflict();

```



### # 加载其他非js资源

如果想要在加载相应的第三方插件时，才一并加载插件对应的样式表，`requireJs`对这种css依赖的加载方式比较麻烦：

```

function loadCss(url) {    

    var link = document.createElement("link");  

    link.type = "text/css";    

    link.rel = "stylesheet";    

    link.href = url;    

    document.getElementsByTagName("head")[0].appendChild(link); 

}

```

这种写法当然不是我第一个觉得不开心，所以一位热心人士贡献了一个插件[requirecss](https://github.com/guybedford/require-css)，现在，你只需要像引入js一样方便对引入你的css以及其他资源：

```
require(['css!my-css', 'image!preload-background-image.jpg', 'font!google,families:[Tangerine]']);

```






