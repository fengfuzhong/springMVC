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
