<h3 align="center"><b>Spring Beans 基础</b></h3>

## 1. Spring `BeanDefinition`

`BeanDefinition`是Spring Framework中定义Bean的配置元信息接口(`org.springframework.beans.factory.config.BeanDefinition`)，包含：

+ Bean的类名(类全名/完全限定名)
+ Bean行为元素(作用域，自动绑定模式，生命周期回调等)
+ 其他Bean引用(依赖dependencies)
+ 配置设置(Bean属性properties)

`BeanDefinition`的构建主要通过以下方式进行：

+ `BeanDefinitionBuilder`
+ `AbstractBeanDefinition`(本身实现了`BeanDefinition`接口)以及子类

`BeanDefinition`的元信息包括：

+ Bean的类全名(必须是具体类，不能是接口或抽象类)：`class`
+ Bean的名称：`id`/`name`
+ Bean的作用域(`singleton`，`prototype`)：`scope`
+ Bean的构造器参数(用于依赖注入)：`constructor-arg`
+ Bean的属性设置(用于依赖注入)：`property`
+ Bean的自动绑定模式(`byName`，`byType`等)：`autowire`
+ Bean的延迟初始化模式(`true`/`false`)：`lazy-init`
+ 初始化回调方法：`init-method`
+ 销毁回调方法：`destroy-method`

### 1.1 通过`BeanDefinitionBuilder`构建