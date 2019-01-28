# 自定义spingMVC
## 一、了解SpringMVC运行流程及九大组件
### 1、SpringMVC的运行流程
![SpringMVC工作原理](https://github.com/fengfuzhong/springMVC/blob/master/src/main/resources/images/SpringMvc%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86%E5%9B%BE.jpg)
      
       ⑴ 用户发送请求至前端控制器DispatcherServlet

       ⑵ DispatcherServlet收到请求调用HandlerMapping处理器映射器。

       ⑶ 处理器映射器根据请求url找到具体的处理器，生成处理器对象及处理器拦截器(如果有则生成)一并返回给DispatcherServlet。

       ⑷ DispatcherServlet通过HandlerAdapter处理器适配器调用处理器

       ⑸ 执行处理器(Controller，也叫后端控制器)。

       ⑹ Controller执行完成返回ModelAndView

       ⑺ HandlerAdapter将controller执行结果ModelAndView返回给DispatcherServlet

       ⑻ DispatcherServlet将ModelAndView传给ViewReslover视图解析器

       ⑼ ViewReslover解析后返回具体View

       ⑽ DispatcherServlet对View进行渲染视图（即将模型数据填充至视图中）。

       ⑾ DispatcherServlet响应用户。
       
### 2、SpringMVC的九大组件
      ⑴ 前端控制器组件（DispatcherServlet） 
      ⑵ 处理器组件（Controller） 
      ⑶ 处理器映射器组件（HandlerMapping） 
      ⑷ 处理器适配器组件（HandlerAdapter） 
      ⑸ 拦截器组件（HandlerInterceptor） 
      ⑹ 视图解析器组件（ViewResolver） 
      ⑺ 视图组件（View） 
      ⑻ 数据转换组件（DataBinder） 
      ⑼ 消息转换器组件（HttpMessageConverter）
###### 以下代码是对DispatcherServlet中的initStrategies方法进行简单描述，该方法调用了DispatcherServlet内部的9个初始化方法，
###### 分别初始化不同的组件：
```
protected void initStrategies(ApplicationContext context) {
 //用于处理上传请求。处理方法是将普通的request包装成MultipartHttpServletRequest，后者可以直接调用getFile方法获取File.
  initMultipartResolver(context);
 //SpringMVC主要有两个地方用到了Locale：一是ViewResolver视图解析的时候；二是用到国际化资源或者主题的时候。
  initLocaleResolver(context); 
 //用于解析主题。SpringMVC中一个主题对应一个properties文件，里面存放着跟当前主题相关的所有资源、
  //如图片、css样式等。SpringMVC的主题也支持国际化， 
 initThemeResolver(context);
 //用来查找Handler的。
 initHandlerMappings(context);
 //从名字上看，它就是一个适配器。Servlet需要的处理方法的结构却是固定的，都是以request和response为参数的方法。
  //如何让固定的Servlet处理方法调用灵活的Handler来进行处理呢？这就是HandlerAdapter要做的事情
 initHandlerAdapters(context);
 //其它组件都是用来干活的。在干活的过程中难免会出现问题，出问题后怎么办呢？
 //这就需要有一个专门的角色对异常情况进行处理，在SpringMVC中就是HandlerExceptionResolver。
 initHandlerExceptionResolvers(context);
 //有的Handler处理完后并没有设置View也没有设置ViewName，这时就需要从request获取ViewName了，
 //如何从request中获取ViewName就是RequestToViewNameTranslator要做的事情了。
 initRequestToViewNameTranslator(context);
 //ViewResolver用来将String类型的视图名和Locale解析为View类型的视图。
 //View是用来渲染页面的，也就是将程序返回的参数填入模板里，生成html（也可能是其它类型）文件。
 initViewResolvers(context);
 //用来管理FlashMap的，FlashMap主要用在redirect重定向中传递参数。
 initFlashMapManager(context); 
}
```
## 自定义SpringMVC框架之前了解的几个知识点：
### 自定义注解（Annotation）
#### 1、认识注解
##### 注解(Annotation)很重要，未来的开发模式都是基于注解的，JPA是基于注解的，Spring2.5以上都是基于注解的，Hibernate3.x以后也是基于注解的，现在的Struts2有一部分也是基于注解的了，注解是一种趋势，现在已经有不少的人开始用注解了，注解是JDK1.5之后才有的新特性。
JDK1.5之后内部提供的三个注解：

       @Deprecated 意思是“废弃的，过时的”
       @Override 意思是“重写、覆盖”
       @SuppressWarnings 意思是“压缩警告”
       
   总结：注解(Annotation)相当于一种标记，在程序中加入注解就等于为程序打上某种标记，没有加，则等于没有任何标记，以后，javac编译器、开发工具和其他程序可以通过反射来了解你的类及各种元素上有无何种标记，看你的程序有什么标记，就去干相应的事，标记可以加在包、类，属性、方法，方法的参数以及局部变量上。
  
 ![注解应用结构图](https://github.com/fengfuzhong/springMVC/blob/master/src/main/resources/images/%E6%B3%A8%E8%A7%A3%E5%BA%94%E7%94%A8%E7%BB%93%E6%9E%84%E5%9B%BE.jpg)
   注解就相当于一个你的源程序要调用一个类，在源程序中应用某个注解，得事先准备好这个注解类。就像你要调用某个类，得事先开发好这个类。
  
 #### 自定义注解及其应用
```
package com.feng.annotation;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
/**
 * 这是一个自定义的注解(Annotation)类 在定义注解(Annotation)类时使用了另一个注解类Retention
 * 在注解类上使用另一个注解类，那么被使用的注解类就称为元注解
 * 
 * @author AlexFeng
 * 
 */
@Retention(RetentionPolicy.RUNTIME)
//Retention注解决定MyAnnotation注解的生命周期
@Target( { ElementType.METHOD, ElementType.TYPE })
//Target注解决定MyAnnotation注解可以加在哪些成分上，如加在类身上，或者属性身上，或者方法身上等成分
/*
 * @Retention(RetentionPolicy.SOURCE)
 * 这个注解的意思是让MyAnnotation注解只在java源文件中存在，编译成.class文件后注解就不存在了
 * @Retention(RetentionPolicy.CLASS)
 * 这个注解的意思是让MyAnnotation注解在java源文件(.java文件)中存在，编译成.class文件后注解也还存在，
 * 被MyAnnotation注解类标识的类被类加载器加载到内存中后MyAnnotation注解就不存在了
 */
/*
 * 这里是在注解类MyAnnotation上使用另一个注解类，这里的Retention称为元注解。
 * Retention注解括号中的"RetentionPolicy.RUNTIME"意思是让MyAnnotation这个注解的生命周期一直程序运行时都存在
 */
public @interface MyAnnotation {
}
```
