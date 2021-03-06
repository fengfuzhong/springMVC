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
## 二、自定义SpringMVC框架之前了解的几个知识点：
### 自定义注解（Annotation）
#### 1、认识注解
##### 注解(Annotation)很重要，未来的开发模式都是基于注解的，JPA是基于注解的，Spring2.5以上都是基于注解的，Hibernate3.x以后也是基于注解的，现在的Struts2有一部分也是基于注解的了，注解是一种趋势，现在已经有不少的人开始用注解了，注解是JDK1.5之后才有的新特性。
JDK1.5之后内部提供的三个注解：

       @Deprecated 意思是“废弃的，过时的”
       @Override 意思是“重写、覆盖”
       @SuppressWarnings 意思是“压缩警告”
       
##### 总结：注解(Annotation)相当于一种标记，在程序中加入注解就等于为程序打上某种标记，没有加，则等于没有任何标记，以后，javac编译器、开发工具和其他程序可以通过反射来了解你的类及各种元素上有无何种标记，看你的程序有什么标记，就去干相应的事，标记可以加在包、类，属性、方法，方法的参数以及局部变量上。
  
 ![注解应用结构图](https://github.com/fengfuzhong/springMVC/blob/master/src/main/resources/images/%E6%B3%A8%E8%A7%A3%E5%BA%94%E7%94%A8%E7%BB%93%E6%9E%84%E5%9B%BE.jpg)
 
#####  注解就相当于一个你的源程序要调用一个类，在源程序中应用某个注解，得事先准备好这个注解类。就像你要调用某个类，得事先开发好这个类。
  
#### 2、自定义注解及其应用
##### 自定义一个最简单的注解：
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
##### 用反射测试进行测试AnnotationUse的定义上是否有@MyAnnotation
```
package com.feng.annotation;
@MyAnnotation
//这里是将新创建好的注解类MyAnnotation标记到AnnotaionTest类上
public class AnnotationUse {
    public static void main(String[] args) {
        // 这里是检查Annotation类是否有注解，这里需要使用反射才能完成对Annotation类的检查
        if (AnnotationUse.class.isAnnotationPresent(MyAnnotation.class)) {
            /*
             * MyAnnotation是一个类，这个类的实例对象annotation是通过反射得到的，这个实例对象是如何创建的呢？
             * 一旦在某个类上使用了@MyAnnotation，那么这个MyAnnotation类的实例对象annotation就会被创建出来了
             */
            MyAnnotation annotation = (MyAnnotation) AnnotationUse.class
                    .getAnnotation(MyAnnotation.class);
            System.out.println(annotation);// 打印MyAnnotation对象，这里输出的结果为：@cn.itcast.day2.MyAnnotation()
        }
    }
}
```
#### 3、@Retention元注解
##### 当在Java源程序上加了一个注解，这个Java源程序要由javac去编译，javac把java源文件编译成.class文件，在编译成class时可能会把Java源程序上的一些注解给去掉，java编译器(javac)在处理java源程序时，可能会认为这个注解没有用了，于是就把这个注解去掉了，那么此时在编译好的class中就找不到注解了， 这是编译器编译java源程序时对注解进行处理的第一种可能情况，假设java编译器在把java源程序编译成class时，没有把java源程序中的注解去掉，那么此时在编译好的class中就可以找到注解，当程序使用编译好的class文件时，需要用类加载器把class文件加载到内存中，class文件中的东西不是字节码，class文件里面的东西由类加载器加载到内存中去，类加载器在加载class文件时，会对class文件里面的东西进行处理，如安全检查，处理完以后得到的最终在内存中的二进制的东西才是字节码，类加载器在把class文件加载到内存中时也有转换，转换时是否把class文件中的注解保留下来，这也有说法，所以说一个注解的生命周期有三个阶段：java源文件是一个阶段，class文件是一个阶段，内存中的字节码是一个阶段,javac把java源文件编译成.class文件时，有可能去掉里面的注解，类加载器把.class文件加载到内存时也有可能去掉里面的注解，因此在自定义注解时就可以使用Retention注解指明自定义注解的生命周期，自定义注解的生命周期是在RetentionPolicy.SOURCE阶段(java源文件阶段)，还是在RetentionPolicy.CLASS阶段(class文件阶段)，或者是在RetentionPolicy.RUNTIME阶段(内存中的字节码运行时阶段)，根据JDK提供的API可以知道默认是在RetentionPolicy.CLASS阶段 (JDK的API写到：the retention policy defaults to RetentionPolicy.CLASS.)

#### 4、@Target元注解
##### @Target元注解决定了一个注解可以标识到哪些成分上，如标识在在类身上，或者属性身上，或者方法身上等成分，@Target默认值为任何元素(成分)
例如：
```
@Target(value={TYPE,FIELD,METHOD,PARAMETER,CONSTRUCTOR,LOCAL_VARIABLE})
@Retention(value=SOURCE)
public @interface SuppressWarnings
//SuppressWarnings是给javac(java编译器)看的，编译器编译完java文件后，@SuppressWarnings注解就没有用了，所以@SuppressWarnings的Retention的属性值是RetentionPolicy.SOURCE

@Target(value=METHOD)
@Retention(value=SOURCE)
public @interface Override
//Override是给javac(java编译器)看的，编译完以后就@Override注解就没有价值了，@Override注解在源代码中有用，编译成.class文件后@Override注解就没有用了，因此@Override的Retention的属性值是RetentionPolicy.SOURCE

@Documented
@Retention(value=RUNTIME)
public @interface Deprecated

```
#### 5、为注解添加属性
    注解可以看成是一种特殊的类，既然是类，那自然可以为类添加属性
##### 语法：类型 属性名();
```
package com.feng.java;
import java.lang.annotation.Target;
import java.lang.annotation.ElementType;

@Target({ElementType.TYPE,ElementType.METHOD})
public @interface MetaAnnotation {
    String value();//元注解MetaAnnotation设置有一个唯一的属性value
}

package com.feng.java;

public enum EumTrafficLamp {
    RED,//红
    YELLOW,//黄
    GREEN//绿
}

package com.feng.java;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE,ElementType.METHOD})
public @interface MyAnnotation {

    String color() default "blue";//为属性指定缺省值
    /**
     * 为注解添加value属性，这个value属性很特殊，如果一个注解中只有一个value属性要设置，
     * 那么在设置注解的属性值时，可以省略属性名和等号不写， 直接写属性值，如@SuppressWarnings("deprecation")，
     * 这里的MyAnnotation注解设置了两个String类型的属性，color和value，
     * 因为color属性指定有缺省值，value属性又是属于特殊的属性，因此使用MyAnnotation注解时
     * 可以这样使用MyAnnotation注解："@MyAnnotation(color="red",value="xdp")"
     * 也可以这样使用："@MyAnnotation("孤傲苍狼")"，这样写就表示MyAnnotation注解只有一个value属性要设置，color属性采用缺省值
     * 当一个注解只有一个value属性要设置时，是可以省略"value="的
     */
    String value();//定义一个名称为value的属性
    //添加一个int类型数组的属性
    int[] arrayAttr() default {1,2,4};
    //添加一个枚举类型的属性，并指定枚举属性的缺省值，缺省值只能从枚举类EumTrafficLamp中定义的枚举对象中取出任意一个作为缺省值
    EumTrafficLamp lamp() default EumTrafficLamp.RED;
    //为注解添加一个注解类型的属性,并指定注解属性的缺省值
    MetaAnnotation annotationAttr() default @MetaAnnotation("xdp");
}

package com.feng.java;

/**
 * 这里是将新创建好的注解类MyAnnotation标记到AnnotaionTest类上，
 * 并应用了注解类MyAnnotation中定义各种不同类型的的属性
 */
@MyAnnotation(
        color="red",
        value="AlexFeng",
        arrayAttr={3,5,6},
        lamp=EumTrafficLamp.GREEN,
        annotationAttr=@MetaAnnotation("gacl")
)
public class MyAnnotationTest {
    @MyAnnotation("将MyAnnotation注解标注到main方法上")
    public static void main(String[] args) {
        /**
         * 这里是检查Annotation类是否有注解，这里需要使用反射才能完成对Annotation类的检查
         */
        if(MyAnnotationTest.class.isAnnotationPresent(MyAnnotation.class)) {
            /**
             * 用反射方式获得注解对应的实例对象后，在通过该对象调用属性对应的方法
             * MyAnnotation是一个类，这个类的实例对象annotation是通过反射得到的，这个实例对象是如何创建的呢？
             * 一旦在某个类上使用了@MyAnnotation，那么这个MyAnnotation类的实例对象annotation就会被创建出来了
             */
            MyAnnotation annotation = (MyAnnotation) MyAnnotationTest.class.getAnnotation(MyAnnotation.class);
            System.out.println(annotation.color());//输出color属性的默认值：red
            System.out.println(annotation.value());//输出value属性的默认值：AlexFeng
            System.out.println(annotation.arrayAttr().length);//这里输出的数组属性的长度的结果为：3，数组属性有三个元素，因此数组的长度为3
            System.out.println(annotation.lamp());//这里输出的枚举属性值为：GREEN
            System.out.println(annotation.annotationAttr().value());//这里输出的注解属性值:gacl

            MetaAnnotation ma = annotation.annotationAttr();//annotation是MyAnnotation类的一个实例对象
            System.out.println(ma.value());//输出的结果为：gacl

        }
    }
}

```

### Servlet(Server Applet)和HttpServlet
#### 1、[Servlet 工作原理解析]（https://www.ibm.com/developerworks/cn/java/j-lo-servlet/）
#### 2、HttpServlet详解
  
    Servlet的框架是由两个Java包组成:javax.servlet和 javax.servlet.http. 在javax.servlet包中定义了所有的Servlet类都必须实现或扩展的的通用接口和类.在javax.servlet.http包中定义了采 用HTTP通信协议的HttpServlet类.
    Servlet的框架的核心是javax.servlet.Servlet接口,所有的Servlet都必须实现这一接口.在Servlet接口中定义了5个方法,其中有3个方法代表了Servlet的声明周期:
    init方法,负责初始化Servlet对象
    service方法,负责相应客户的请求
    destory方法,当Servlet对象退出声明周期时,负责释放占有的资源
    
    若web.xml配置文件中<load-on-startup>这个参数的值大于等于0，则是在tomact一启动，则new出来。并且值越小，优先级越高，优先被实例化。若参数的值为负数，则是在输入url时new出来，而在tomcat启动时无任何反应。若是没有配置此参数，相当于参数值为负数。即在输入url运行时才new出来。Servlet在被创建出来实例化时会先调用init（），在服务器停止的时候会调用destroy()。通过继承HttpServlet这个类，并在其子类中重写init()和destroy()方法，就能根据自己的要求在服务器tomcat起来时和结束时运行一些自己的代码逻辑.
    当Web容器接收到某个Servlet请求时,Servlet把请求封装成一个HttpServletRequest对象,然后把对象传给Servlet的对应的服务方法.
         HTTP的请求方式包括DELETE,GET,OPTIONS,POST,PUT和TRACE,在HttpServlet类中分别提供了相应的服务方法, 它们是,doDelete(),doGet(),doOptions(),doPost(), doPut()和doTrace(). 
         
##### HttpServlet的功能
    HttpServlet首先必须读取Http请求的内容。Servlet容器负责创建HttpServlet对象，并把Http请求直接封装到 HttpServlet对象中，大大简化了HttpServlet解析请求数据的工作量。HttpServlet容器响应Web客户请求流程如下：
    （1）Web客户向Servlet容器发出Http请求；
    （2）Servlet容器解析Web客户的Http请求；
    （3）Servlet容器创建一个HttpRequest对象，在这个对象中封装Http请求信息；
    （4）Servlet容器创建一个HttpResponse对象；
    （5）Servlet容器调用HttpServlet的service方法，把HttpRequest和HttpResponse对象作为service方法的参数传给HttpServlet对象；
    （6）HttpServlet调用HttpRequest的有关方法，获取HTTP请求信息；
    （7）HttpServlet调用HttpResponse的有关方法，生成响应数据；
    （8）Servlet容器把HttpServlet的响应结果传给Web客户。

## 自定义SpringMvc框架
### 设计思路
#### 1、读取配置
    SpringMVC本质上是一个Servlet,这个 Servlet 继承自 HttpServlet。FrameworkServlet负责初始化SpringMVC的容器，并将Spring容器设置为父容器。
    为了读取web.xml中的配置，我们用到ServletConfig这个类，它代表当前Servlet在web.xml中的配置信息。通过web.xml中加载我们自己写的MyDispatcherServlet和读取配置文件。
   
#### 2、初始化阶段
    在前面我们提到DispatcherServlet的initStrategies方法会初始化9大组件，但是这里将实现一些SpringMVC的最基本的组件而不是全部，按顺序包括：
    （1）加载配置文件，在application.properties文件中配置了需要扫描到SpringMV容器中的包名
    （2）扫描用户配置包下面所有的类。一般特指controller类，通过判断类注解上是否包含MyController类
    （3）拿到扫描到的类，通过反射机制，实例化。并且放到ioc容器中(Map的键值对 beanName-bean) beanName默认是首字母小写
    （4）初始化HandlerMapping，这里其实就是把url和method对应起来放在一个k-v的Map中,在运行阶段取出
    
#### 4、运行阶段
    每一次请求将会调用doGet或doPost方法，所以统一运行阶段都放在doDispatch方法里处理，它会根据url请求去HandlerMapping中匹配到对应的Method，然后利用反射机制调用Controller中的url对应的方法，并得到结果返回。按顺序包括以下功能：
    （1）异常的拦截
    （2）获取请求传入的参数并处理参数
    （3）通过初始化好的handlerMapping中拿出url对应的方法名，反射调用
   
### 工程结构
![自定义SpringMVC框架示意图](https://github.com/fengfuzhong/springMVC/blob/master/src/main/resources/images/SpringMVC%E6%A1%86%E6%9E%B6%E7%BB%93%E6%9E%84%E5%9B%BE.png)
