### mvc对静态资源的访问

1. 在springmvc-servlet.xml配置文件添加

   ```xml
   <mvc:default-servlet-handler/>
   ```

   ​

2. 在web.xml添加如下配置

   1. ```xml
      <servlet-mapping>
        <servlet-name>default</servlet-name>
        <url-pattern>*.js</url-pattern>
        <url-pattern>*.css</url-pattern>
        <url-pattern>/images/*</url-pattern>
      </servlet-mapping>
      ```


### 自定义bean validation注解

```java
package com.controller;

import javax.validation.Constraint;
import javax.validation.Payload;
import java.lang.annotation.*;

import static java.lang.annotation.ElementType.*;
import static java.lang.annotation.ElementType.PARAMETER;
import static java.lang.annotation.RetentionPolicy.RUNTIME;

/**
 * 自定义参数校验注解
 * 校验 username
 */

@Target({ElementType.ANNOTATION_TYPE, ElementType.METHOD, ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Constraint(validatedBy = UsernameExistImpl.class)////此处指定了注解的实现类为UsernameExistImpl

public @interface UsernameExist{

    /**
     * 添加value属性，可以作为校验时的条件,若不需要，可去掉此处定义
     */
    int value() default 0;

    String message() default "用户名test已经存在";

    Class<?>[] groups() default {};

    Class<? extends Payload>[] payload() default {};

    /**
     * 定义一个新的名称，为了让Bean的一个属性上可以添加多套规则
     */
    @Target({METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER})
    @Retention(RUNTIME)
    @Documented
    @interface UsernameExists{
        UsernameExist[] value();
    }
}
```

```java
package com.controller;
import org.springframework.stereotype.Service;

import javax.validation.ConstraintValidator;
import javax.validation.ConstraintValidatorContext;

/**
 * 自定义注解UsernameExist 的实现类
 * 
 */

@Service
public class UsernameExistImpl implements ConstraintValidator<UsernameExist, String> {

    private int value;

    @Override
    public void initialize(UsernameExist constraintAnnotation) {
        //传入value 值，可以在校验中使用
        this.value = constraintAnnotation.value();
    }

    @Override
    public boolean isValid(String o, ConstraintValidatorContext constraintValidatorContext) {
        System.out.println("++++++++++"+o.toString());
        return  o.toString().equals("test");
    }

}
```
问题描述：springMVC No mapping found for HTTP request with URI
原因：
－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－
No mapping found for HTTP request with URI
出现这个问题的原因是在web.xml中配置错了，如：
```xml
 <servlet>
 <servlet-name>springMVCDispatcherServlet</servlet-name>
 <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
 <init-param>
 <param-name>contextConfigLocation</param-name>
 <param-value>/WEB-INF/springMVC-servlet.xml</param-value>
 </init-param>
  <load-on-startup>1</load-on-startup>
 </servlet>
 <servlet-mapping>
 <servlet-name>springMVCDispatcherServlet</servlet-name>
 <url-pattern>/*</url-pattern>
 </servlet-mapping>
```
当你在control中返回一个路径的时候，它又把路径（/view/index.jsp）当作一个请求被dispatcherServlet所拦截。所以会抛出异常，解决的办法有两个：
第一即使让dispatcherServlet的拦截加上后缀如：*.do;
这样以jsp后缀的就不会别拦截了。
第二个方法是在spring-servlet.xml中加入：
```xml
<mvc:default-servlet-handler/>
```

就解决了此问题



**415 Unsupported Media Type** 

```javascript
 $.ajax( {  
        type : "post",  
        url : "getJSONString2.zcy?t=" + new Date().getTime(),  
        data : jsonString,  
        contentType : "application/json",   
          
    });  
```

