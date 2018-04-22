```java
Map<String, T> getBeansOfType(Class<T> type) //根据指定的类型返回一个键值为名字和值为Bean对象的 Map，如果没有Bean对象存在则返回空的Map
```

```xml
<import resource=”resource.xml”/>
import用于导入其他配置文件的Bean定义，这是为了加载多个配置文件，当然也可以把这些配置文件构造为一个数组（new String[] {“config1.xml”, config2.xml}）传给ApplicationContext实现进行加载多个配置文件，那一个更适合由用户决定；这两种方式都是通过调用Bean Definition Reader 读取Bean定义，内部实现没有任何区别。<import>标签可以放在<beans>下的任何位置，没有顺序关系。
```

```java
 @PostConstruct
//注解在方法上，实例化bean对象后，执行改方法
@PreDestroy
//注解在方法上，销毁bean对象后，执行改方法

```

**@Value 可以取得配置文件内容**

jdbc.properties配置文件片段

```properties
#定义最大空闲
maxIdle=20
```

bean xml配置文件片段

```xml
 <bean id="propertyConfigurer" 			  class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
        <property name="location" value="classpath:jdbc.properties" />
 </bean>
```
java代码片段
```java
@Autowired(required = true)
@Value("${maxIdle}")
public void setValue(int value) {
    this.value = value;
}
```

**@Component**

```java
/**
 * 相当于在bean.xml配置<bean id="intValuePro" class="study.IntValuePro"/>
 */
@Component("intValuePro")
public class IntValuePro {
}
```

**使用p命名空间简化setter注入：**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans  xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:p="http://www.springframework.org/schema/p"
        xsi:schemaLocation="
           http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">
<bean id="bean1" class="java.lang.String">
        <constructor-arg index="0" value="test"/>
    </bean>
<bean id="idrefBean1" class="cn.javass.spring.chapter3.bean.IdRefTestBean" 
p:id="value"/>
<bean id="idrefBean2" class="cn.javass.spring.chapter3.bean.IdRefTestBean" 
p:id-ref="bean1"/>
</beans>

xmlns:p="http://www.springframework.org/schema/p" ：首先指定p命名空间；
<bean id="……" class="……" p:id="value"/> ：常量setter注入方式，其等价于<property name="id" value="value"/>；
<bean id="……" class="……" p:id-ref="bean1"/> ：引用setter注入方式，其等价于<property name="id" ref="bean1"/>。
```

**修改SpEL前后缀**

使用BeanFactoryPostProcessor接口提供postProcessBeanFactory回调方法，它是在IoC容器创建好但还未进行任何Bean初始化时被ApplicationContext实现调用，因此在这个阶段把SpEL前缀及后缀修改掉是安全的，具体代码如下：

```java
import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.BeanFactoryPostProcessor;
import org.springframework.beans.factory.config.ConfigurableListableBeanFactory;
import org.springframework.context.expression.StandardBeanExpressionResolver;

public class SpELBeanFactoryPostProcessor implements BeanFactoryPostProcessor {
    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory)
        throws BeansException {
        StandardBeanExpressionResolver resolver =(StandardBeanExpressionResolver) beanFactory.getBeanExpressionResolver();
        resolver.setExpressionPrefix("%{");
        resolver.setExpressionSuffix("}");
    }
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans  xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:context="http://www.springframework.org/schema/context"
        xsi:schemaLocation="
           http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
           http://www.springframework.org/schema/context
           http://www.springframework.org/schema/context/spring-context-3.0.xsd">
   <context:annotation-config/>
   <bean class="cn.javass.spring.chapter5.SpELBeanFactoryPostProcessor"/>
</beans>
```

