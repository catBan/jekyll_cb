---
layout: post
title:  "拓展@ResponseBody能convert的类型"
date:   2017-01-16 21:29:01 +0800
categories: 学无止境
tag: Enheng
---

* content
{:toc}

> 事情的起因是要做一个前后端统一的验证框架，当进行到前端请求远程验证,有需要返回true/false时，遇到一个问题：由于这个验证方法在后端验证时也会使用，为了方便开发，不想把返回值的类型限制为`String`，想同时支持`Boolean`或者`String`;但是SpringMVC中的`@ResponseBody` 不支持Boolean类型的返回值



### 问题：解决[对象－报文]无法转换

错误日志中提示的是无法convert这个返回类型；由此想到可以自定义自己的converter。

能先入手的关键词只有`@ResponseBody`，查看doc可以看到一段关键信息：

```


If the method is annotated with @ResponseBody, the return type is written to the response HTTP body. The return value will be converted to the declared method argument type using HttpMessageConverters.

```

这样一来，解决的切入点定位到 `HttpMessageConverters`。查看 `HttpMessageConverters`这个接口会发现，这其实是一个`@RequestBody`/`@ResponseBody`所完成的报文－对象间的转换过程：

```

T read(Class<? extends T> clazz, HttpInputMessage inputMessage)
     throws IOException, HttpMessageNotReadableException;

void write(T t, MediaType contentType, HttpOutputMessage outputMessage)
     throws IOException, HttpMessageNotWritableException;

```

到这里就清晰多了，其中的pair `HttpInputMessage`/`HttpOutputMessage` 是对请求报文／响应报文的一层内部抽象，也就是我最后要拓展的返回类型要操作的最终对象。

盗一个图说明一下这其中的关系：

![whats]({{ '/styles/images/post/17011701.png' | prepend: site.baseurl  }})






跟踪代码可以发现，被@ResponseBody注解的方法，在返回数据报文时，会去找到适当的`HttpMessageConverters`实现类来解析返回值。

Spring默认了提供了适用于大部分情况的converter：

```

ByteArrayHttpMessageConverter

StringHttpMessageConverter

MappingJackson2HttpMessageConverter

...

等等

```

### 实现自己的converter

`BooleanHttpMessageConverter`继承`AbstractHttpMessageConverter`，重写几个方法:

```


@Override
protected boolean supports(Class<?> clazz) {
   return Boolean.class == clazz;
}

/*readInternal:从http报文中读取信息，转换成自己想要的类型 */
@Override
protected Boolean readInternal(Class<? extends Boolean> clazz, HttpInputMessage inputMessage) throws IOException, HttpMessageNotReadableException {
   String temp = StreamUtils.copyToString(inputMessage.getBody(),getContentTypeCharset(inputMessage.getHeaders().getContentType()));
   return BooleanUtils.toBoolean(temp);
}

/*writeInternal:将想返回的对象，转换成客户端可接收的报文类型返回 */

@Override
protected void writeInternal(Boolean aBoolean, HttpOutputMessage outputMessage) throws IOException, HttpMessageNotWritableException {
   Charset charset = getContentTypeCharset(outputMessage.getHeaders().getContentType());
   StreamUtils.copy(BooleanUtils.toStringTrueFalse(aBoolean),charset,outputMessage.getBody());
}

```







### 将custom messageConverter 注册到springBoot注册的messageConver中

完成了自己的converter后接下来就是要将注册converter。、

跟踪代码，会发现查找适合到converter的过程在`AbstractMessageConverterMethodProcessor`的

`writeWithMessageConverters`方法中,

```

protected List<MediaType> getProducibleMediaTypes(HttpServletRequest request, Class<?> valueClass, Type declaredType) {
  Set<MediaType> mediaTypes = (Set<MediaType>) request.getAttribute(HandlerMapping.PRODUCIBLE_MEDIA_TYPES_ATTRIBUTE);
  if (!CollectionUtils.isEmpty(mediaTypes)) {
     return new ArrayList<MediaType>(mediaTypes);
  }
  else if (!this.allSupportedMediaTypes.isEmpty()) {
     List<MediaType> result = new ArrayList<MediaType>();
     for (HttpMessageConverter<?> converter : this.messageConverters) {
        if (converter instanceof GenericHttpMessageConverter && declaredType != null) {
           if (((GenericHttpMessageConverter<?>) converter).canWrite(declaredType, valueClass, null)) {
              result.addAll(converter.getSupportedMediaTypes());
           }
        }
        else if (converter.canWrite(valueClass, null)) {
           result.addAll(converter.getSupportedMediaTypes());
        }
     }
     return result;
  }
  else {
     return Collections.singletonList(MediaType.ALL);
  }
}
```

其中注册的关键点，就是那个`messageConverters`。

#### 1.重写`WebMvcConfigurerAdapter`中的`extendMessageConverters`

```

public class MyWebMvcConfigurer extends WebMvcConfigurerAdapter {

   @Override
   public void extendMessageConverters(List<HttpMessageConverter<?>> converters){
       converters.clear();
       converters.add(new BooleanHttpMessageConverter());
   }
}

```

#### 2.注册

在xml中配置或可通过注解配置，我用的是xml配置，注解没去实践

```

<mvc:annotation-driven content-negotiation-manager="contentNegotiationManager">
   <mvc:message-converters register-defaults="true">
       <bean class="com.catban.common.tool.BooleanHttpMessageConverter"/>
   </mvc:message-converters>
</mvc:annotation-driven>

<bean id="contentNegotiationManager"
     class="org.springframework.web.accept.ContentNegotiationManagerFactoryBean">
   <property name="mediaTypes">
       <props>
           <prop key="json">application/json</prop>
           <prop key="xml">application/xml</prop>
       </props>
   </property>
</bean>

```



### 自定义完成

```

   @RequestMapping(value = "/checkPwdStrength",method = {RequestMethod.POST})
   @ResponseBody
   public Boolean checkPwdStrength(@RequestParam("userPass") String userPass) {
       if (!userPass.matches(RegexpConstant.PWD_STRENGTH)) {
           return false;
       }
       return true;
   }

```

### 结语

虽然解决的问题不是什么大问题，写的条理也不是很清晰，但不论是解决问题的过程还是写下来的过程，多少对自己也是有收获跟锻炼。想着以后能更深入的解决其他问题时，也才能慢慢知道该怎么记录才清晰  现在就像流水账一样哈


