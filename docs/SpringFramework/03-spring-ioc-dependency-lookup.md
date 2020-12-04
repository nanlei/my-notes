<h3 align="center"><b>Spring IoC 依赖查找</b></h3>

依赖查找可以分为：

+ 单一类型依赖查找
    +  JNDI - javax.naming.Context#lookup(javax.naming.Name)
    +  JavaBeans - java.beans.beancontext.BeanContext
+ 集合类型依赖查找
    + java.beans.beancontext.BeanContext
+ 层次性依赖查找
    + java.beans.beancontext.BeanContext

`BeanContext`规范，[可以参考](https://docs.oracle.com/javase/8/docs/technotes/guides/beans/spec/beancontext.html)

## 1. Spring IoC 单一类型依赖查找

单一类型依赖查找接口：`BeanFactory`

+ 根据`Bean`名称查找
    + `getBean(String name)`
    + Spring 2.5覆盖默认参数：`getBean(String name, Object... args)`
+ 根据`Bean`类型查找
    + 实时查找
        + Spring 3.0：`getBean(Class<T> requiredType)`
        + Spring 4.1覆盖默认参数：`getBean(Class<T> requiredType, Object... args)`
    + 延迟查找(Spring 5.1)
        + `getBeanProvider(Class<T> requiredType)`
        + `getBeanProvider(ResolvableType requiredType)`
+ 根据`Bean`名称+类型查找
    + `getBean(String name, Class<T> requiredType)`

`getBeanProvider`依赖查找示例：
```java
package deep.in.spring.dependency.lookup;

import org.springframework.beans.factory.ObjectProvider;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Bean;

/**
 * 通过 {@link ObjectProvider} 进行依赖查找
 */
public class ObjectProviderDemo { // @Configuration是非必须的注解
    public static void main(String[] args) {
        //构建 ApplicationContext 容器
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext();
        //注册 Configuration 类(配置类)
        applicationContext.register(ObjectProviderDemo.class);
        //启动 Spring 应用上下文
        applicationContext.refresh();
        //依赖查找对象
        lookupByObjectProvider(applicationContext);
        //关闭 Spring 应用上下文
        applicationContext.close();
    }

    @Bean
    public String helloworld() {//方法名就是 Bean 名称: helloworld
        return "Hello,World";
    }

    private static void lookupByObjectProvider(AnnotationConfigApplicationContext applicationContext) {
        ObjectProvider<String> objectProvider = applicationContext.getBeanProvider(String.class);
        System.out.println(objectProvider.getObject());
    }
}
```

## 2. Spring IoC 集合类型依赖查找

集合类型依赖查找接口：`ListableBeanFactory`

+ 根据`Bean`类型查找
    + 获取同类型`Bean`名称列表
        + `getBeanNamesForType(@Nullable Class<?> type)`
        + Spring 4.2：`getBeanNamesForType(ResolvableType type)`
    + 获取同类型`Bean`实例列表
        + `getBeansOfType(@Nullable Class<T> type)`
+ 通过注解类型查找
    + 获取标注类型`Bean`名称列表
       + `getBeanNamesForAnnotation(Class<? extends Annotation> annotationType)`
   + 获取标注类型`Bean`实例列表
       + `getBeansWithAnnotation(Class<? extends Annotation> annotationType)`
   + 获取指定名称+标注类型`Bean`实例
       + `findAnnotationOnBean(String beanName, Class<A> annotationType)`

## 3. Spring IoC 层次性依赖查找

层次性依赖查找接口：`HierarchicalBeanFactory`

+ 双亲`BeanFactory`：getParentBeanFactory()
+ 层次性查找
    + 根据`Bean`名称查找
        + `containsLocalBean(String name)`
    + 根据`Bean`类型查找实例列表
        + 单一类型：`BeanFactoryUtils#beanOfType`
        + 集合类型：`BeanFactoryUtils#beansOfTypeIncludingAncestors`
    + 根据Java注解查找名称列表
        + `BeanFactoryUtils#beanNamesForTypeIncludingAncestors`

