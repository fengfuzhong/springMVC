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
