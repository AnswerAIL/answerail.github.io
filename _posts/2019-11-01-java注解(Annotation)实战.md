---
layout: post
title: java注解(Annotation)实战
tags: [开发笔记]
---

## Java 预定义注解

 Java 支持一组预先定义好的注解。下面介绍了Java Core 中提供的注解 

- @Retention： 该注解用来修饰其他注解，并标明被修饰注解的作用域。其 value 的属性值包含3种：
  - SOURCE： 注解仅在源代码中可用。编译器和 JVM 会忽略此注解，因此在运行时不可用；
  - CLASS： 编译器会处理该注解，但 JVM 不会处理，因此在运行时不可用；
  - RUNTIME： JVM 会处理该注解，可以在运行时使用。
- @Target： 该注解标记可以应用的目标元素：
  - ANNOTATION_TYPE： 可修饰其他注解；
  - CONSTRUCTOR： 可以修饰构造函数；
  - FIELD： 可以修饰字段或属性；
  - LOCAL_VARIABLE： 可以修饰局部变量；
  - METHOD： 可以修饰 method；
  - PACKAGE： 可以修饰 package 声明；
  - PARAMETER： 可以修饰方法参数；
  - TYPE： 可以修饰 Class、Interface、Annotation 或 enum 声明；
  - PACKAGE： 可以修饰 package 声明；
  - TYPE_PARAMETER： 可以修饰参数声明；
  - TYPE_USE： 可以修饰任何类型。
- @Documented： 该注解可以修饰其他注解，表示将使用 Javadoc 记录被注解的元素。
- @Inherited： 默认情况下，注解不会被子类继承。但是，如果把注解标记为 @Inherited，那么使用注解修饰 class 时，子类也会继承该注解。该注解仅适用于 class。注意：使用该注解修饰接口时，实现类不会继承该注解。
- @Deprecated： 标明不应该使用带此注解的元素。使用这个注解，编译器会对应生成告警。该注解可以应用于 method、class 和字段。
- @SuppressWarnings： 告诉编译器由于特定原因不产生告警。
- @Override： 该注解通知编译器，该元素正在覆盖（Override）父类中的元素。覆盖元素时，不强制要求加上该注解。但是当覆盖没有正确完成时，例如子类方法的参数与父类参数不同或者返回类型不匹配时，可以帮助编译器生成错误。
- @SafeVarargs：该注解断言（Assert）方法或构造函数代码不会对其参数执行不安全（Unsafe）操作。
-  @Repeatable： 该注解表示可以对同一个元素多次使用相同的注解 

&nbsp;



## Repeatable 实战
定义一个可重复修饰 class 的注解

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Repeatable(RepeatableAnnotation.class)
public @interface MyAnnotation {
    String value();

    String name();
}
```

>  RepeatableAnnotation可以重复修饰元素 



```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface RepeatableAnnotation {
    MyAnnotation[] value();
}
```



```java
@MyAnnotation(value = "Jaemon", name = "J.M")
@MyAnnotation(value = "Answer", name = "L.A")
public class AnswerApp {

    public static void main(String[] args) {
        for (Annotation annotation : AnswerApp.class.getAnnotations()) {
            System.out.println(annotation.toString());
        }
        System.out.println();

        RepeatableAnnotation annotation = AnswerApp.class.getAnnotation(RepeatableAnnotation.class);
        for (MyAnnotation myAnnotation : annotation.value()) {
            System.out.println(myAnnotation.value());
            System.out.println(myAnnotation.name());
        }
    }

}
```



```java
@com.answer.ai.anno.RepeatableAnnotation(value=[@com.answer.ai.anno.MyAnnotation(value=Jaemon, name=J.M), @com.answer.ai.anno.MyAnnotation(value=Answer, name=L.A)])

Jaemon
J.M
Answer
L.A
```

&nbsp;



## 参考网址

- [x] [Java 8 注解探秘](https://mp.weixin.qq.com/s?__biz=MjM5NzMyMjAwMA==&mid=2651485369&idx=1&sn=d8ec5bf115b25874704335e09496c2cf&chksm=bd2518c68a5291d0f40a955478410432f99abc2386152b8abb230b4adc100b3f26f8719ef89b&scene=0&xtrack=1&key=62dc85f35374a3100299f0b2636f96e097c8335673b592aa0fa21999a1c3a838dcff063dea377b3ec3fd3b2c65445f7696964467b2eeadc410d6bb9c8561bc8394bfd6e8894d3b72649309384c2ecea0&ascene=1&uin=MTQ2NzE4ODQ2NA%3D%3D&devicetype=Windows+10&version=62060833&lang=zh_CN&pass_ticket=Ggba1Wee%2BgCpTUxAetZPOen0UEMIW29c3WapA0htyZObG9cBGRNvmcuy0Tsy5PgQ)