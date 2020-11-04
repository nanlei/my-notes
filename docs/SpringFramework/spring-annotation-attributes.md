<h3 align="center"><b>Spring 注解属性</b></h3>

在Java语言中，注解之间没有继承关系，也不能实现接口，但Java默认所有注解实现`Annotation`接口：
```java
package java.lang.annotation;

public interface Annotation {
    boolean equals(Object obj);

    int hashCode();

    String toString();

    Class<? extends Annotation> annotationType();
}
```

被标注的对象可以通过`AnnotatedElement` API进行表达，`<T extends Annotation> T getAnnotation(Class<T> annotationClass)`方法返回指定类的注解对象，获取注解属性即可调用对应的属性方法。

## 1.理解`AnnotationMeta`

以`@TransactionalService`为例，增加属性方法`name()`，表示Bean名称，默认为空字符串：
```java
package deep.in.springframework.annotation;

import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.lang.annotation.*;

@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Transactional
@Service
public @interface TransactionalService {
    String name() default "";
}
```

在Java反射中，表示类的`Class`类型定义为`public final class Class<T> implements java.io.Serializable,GenericDeclaration,Type,AnnotatedElement`，也就是说`Class`本身实现了`AnnotatedElement`接口，那么就可以利用Java反射机制来获取`@TransactionalService`属性方法`name()`的内容：
```java
package deep.in.springframework;

import deep.in.springframework.annotation.TransactionalService;

import java.lang.reflect.AnnotatedElement;

@TransactionalService(name = "test")
public class AnnotationReflectionBootstrap {
    public static void main(String[] args) {
        //Class实现了AnnotatedElement接口
        AnnotatedElement annotatedElement = AnnotationReflectionBootstrap.class;
        //从AnnotatedElement获取TransactionalService
        TransactionalService transactionalService = annotatedElement.getAnnotation(TransactionalService.class);
        //显式调用属性方法获取内容
        String nameAttribute = transactionalService.name();
        System.out.println("TransactionalService.name() = " + nameAttribute);
    }
}
```

运行程序即可得到结果：
```txt
TransactionalService.name() = test
```

结果来源于`@TransactionalService(name = "test")`语句，但以上实现并没有完全使用反射，获取name属性是显式调用`@TransactionalService.name()`获取的，修改代码完全使用反射：
```java
package deep.in.springframework;

import deep.in.springframework.annotation.TransactionalService;

import java.lang.reflect.AnnotatedElement;
import java.lang.reflect.Method;

@TransactionalService(name = "test")
public class AnnotationReflectionBootstrap {
    public static void main(String[] args) throws Exception {
        //Class实现了AnnotatedElement接口
        AnnotatedElement annotatedElement = AnnotationReflectionBootstrap.class;
        //从AnnotatedElement获取TransactionalService
        TransactionalService transactionalService = annotatedElement.getAnnotation(TransactionalService.class);
        //Java反射实现
        Class clz = TransactionalService.class;
        Method method = clz.getMethod("name");
        Object result = method.invoke(transactionalService);
        System.out.printf("@TransactionalService.%s() = %s\n", method.getName(), result);
    }
}
```

运行程序即可得到结果：
```txt
@TransactionalService.name() = test
```

也可以使用Spring Framework提供的`ReflectionUtils`工具类来实现：
```java
        //Java反射实现(ReflectionUtils为Spring反射工具类)
        ReflectionUtils.doWithMethods(TransactionalService.class,
                method -> System.out.printf("@TransactionalService.%s() = %s\n", method.getName(), ReflectionUtils.invokeMethod(method, transactionalService)),//执行method调用
                method -> method.getParameterCount() == 0);//ReflectionUtils.invokeMethod(method, transactionalService)决定是无参数方法
```

再次运行程序，可以得到：
```txt
@TransactionalService.name() = test
@TransactionalService.toString() = @deep.in.springframework.annotation.TransactionalService(name=test)
@TransactionalService.hashCode() = 431984231
@TransactionalService.annotationType() = interface deep.in.springframework.annotation.TransactionalService
```

这里`ReflectionUtils.invokeMethod(method, transactionalService)`的写法表示的是无参数方法，所以第三个参数要设置为过滤出无参数的方法。

因为前面提到所有注解都默认实现了`Annotation`接口，因此结果中也包含`Annotation`接口的无参数方法。

若需要将`Annotation`接口的方法予以剔除，可以修改为：
```java
        ReflectionUtils.doWithMethods(TransactionalService.class,
                method -> System.out.printf("@TransactionalService.%s() = %s\n", method.getName(), ReflectionUtils.invokeMethod(method, transactionalService)),//执行method调用
                method -> !method.getDeclaringClass().equals(Annotation.class));
```

`@TransactionalService`元标注了`@Transactional`和`@Service`注解，在`@Transactional`中也有不少方法：
```java
public @interface Transactional {

	...
	@AliasFor("transactionManager")
	String value() default "";

	...
	@AliasFor("value")
	String transactionManager() default "";

	...
	Propagation propagation() default Propagation.REQUIRED;

	...
	Isolation isolation() default Isolation.DEFAULT;

	...
	int timeout() default TransactionDefinition.TIMEOUT_DEFAULT;

	...
	boolean readOnly() default false;

	...
	Class<? extends Throwable>[] rollbackFor() default {};

	...
	String[] rollbackForClassName() default {};

	...
	Class<? extends Throwable>[] noRollbackFor() default {};

	...
	String[] noRollbackForClassName() default {};

}
```

用Java反射描述`@Transactional`，因为其也是标注在`@TransactionalService`上的`Annotation`对象，所以`Transactional.class`也是`AnnotatedElement`对象，需要再次调用`getAnnotation(Class<T> annotationClass)`方法获取`@Transactional`对象，继续修改代码：
```java
package deep.in.springframework;

import deep.in.springframework.annotation.TransactionalService;
import org.springframework.util.ObjectUtils;
import org.springframework.util.ReflectionUtils;

import java.lang.annotation.Annotation;
import java.lang.reflect.AnnotatedElement;
import java.util.Collections;
import java.util.HashSet;
import java.util.Set;
import java.util.stream.Collectors;
import java.util.stream.Stream;

@TransactionalService(name = "test")
public class AnnotationReflectionBootstrap {
    public static void main(String[] args) throws Exception {
        //Class实现了AnnotatedElement接口
        AnnotatedElement annotatedElement = AnnotationReflectionBootstrap.class;
        //从AnnotatedElement获取TransactionalService
        TransactionalService transactionalService = annotatedElement.getAnnotation(TransactionalService.class);
        //获取transactionalService的所有元注解
        Set<Annotation> metaAnnotations = getAllMetaAnnotations(transactionalService);
        //打印结果
        metaAnnotations.forEach(AnnotationReflectionBootstrap::printAnnotationAttributes);
    }

    private static Set<Annotation> getAllMetaAnnotations(Annotation annotation) {
        Annotation[] metaAnnotations = annotation.annotationType().getAnnotations();

        //没有找到，返回空集合
        if (ObjectUtils.isEmpty(metaAnnotations)) {
            return Collections.emptySet();
        }

        //获取所有非Java标准的元注解集合(通过java.lang.annotation包名排除，因为@Target，@Documented等相互依赖，会导致无限递归)
        Set<Annotation> metaAnnotationsSet = Stream.of(metaAnnotations)
                .filter(metaAnnotation -> !Annotation.class.getPackage().equals(metaAnnotation.annotationType().getPackage()))
                .collect(Collectors.toSet());

        //递归查找元注解的元注解集合
        Set<Annotation> metaMetaAnnotationsSet = metaAnnotationsSet.stream()
                .map(AnnotationReflectionBootstrap::getAllMetaAnnotations)
                .collect(HashSet::new, Set::addAll, Set::addAll);

        //添加递归结果
        metaAnnotationsSet.addAll(metaMetaAnnotationsSet);

        return metaAnnotationsSet;
    }

    private static void printAnnotationAttributes(Annotation annotation) {
        Class<?> annotationType = annotation.annotationType();

        ReflectionUtils.doWithMethods(annotationType,
                method -> System.out.printf("@%s.%s() = %s\n", annotationType.getSimpleName(), method.getName(), ReflectionUtils.invokeMethod(method, annotation)),
                method -> !method.getDeclaringClass().equals(Annotation.class));
    }
}
```

运行程序，可以得到：
```txt
@Service.value() = 
@Component.value() = 
@Transactional.transactionManager() = 
@Transactional.propagation() = REQUIRED
@Transactional.isolation() = DEFAULT
@Transactional.rollbackFor() = [Ljava.lang.Class;@736e9adb
@Transactional.rollbackForClassName() = [Ljava.lang.String;@6d21714c
@Transactional.noRollbackFor() = [Ljava.lang.Class;@108c4c35
@Transactional.noRollbackForClassName() = [Ljava.lang.String;@4ccabbaa
@Transactional.value() = 
@Transactional.readOnly() = false
@Transactional.timeout() = -1
```

结果中包含`@TransactionalService`所有元注解的属性方法值。基于反射获取元注解及属性信息的实现依赖递归查找。而从Spring Framework 4.0.x开始，`AnnotationMetadata`的`getMetaAnnotationTypes(String annotationName)`方法可以获取到所有元注解的类型集合。

`AnnotationMetadata`有两种实现方式，基于`ASM`的`AnnotationMetadataReadingVisitor`(Spring Framework 5.2开始在其内部使用`SimpleAnnotationMetadataReadingVisitor`代替)和基于反射的`StandardAnnotationMetadata`。

使用`StandardAnnotationMetadata`重构上述代码：
```java
package deep.in.springframework;

import deep.in.springframework.annotation.TransactionalService;
import org.springframework.core.type.AnnotationMetadata;
import org.springframework.util.ClassUtils;
import org.springframework.util.CollectionUtils;

import java.util.LinkedHashSet;
import java.util.Map;
import java.util.Set;

@TransactionalService
public class StandardAnnotationMetadataBootstrap {
    public static void main(String[] args) {
        //读取@TransactionalService的AnnotationMetadata
        //Spring Framework 5.2开始不直接使用new StandardAnnotationMetadata(Class<?> introspectedClass)
        //AnnotationMetadata.introspect(Class<?> type)方法使用StandardAnnotationMetadata.from(Class<?> introspectedClass)
        //StandardAnnotationMetadata.from(Class<?> introspectedClass)调用new StandardAnnotationMetadata(introspectedClass, true)
        AnnotationMetadata annotationMetadata = AnnotationMetadata.introspect(StandardAnnotationMetadataBootstrap.class);
        //获取所有的元注解类型(类全名)的集合
        Set<String> metaAnnotationTypes = annotationMetadata.getAnnotationTypes().stream()
                .map(annotationMetadata::getMetaAnnotationTypes)
                .collect(LinkedHashSet::new, Set::addAll, Set::addAll);

        metaAnnotationTypes.forEach(metaAnnotation -> {
            //获取元注解属性信息
            Map<String, Object> annotationAttributes = annotationMetadata.getAnnotationAttributes(metaAnnotation);
            if (!CollectionUtils.isEmpty(annotationAttributes)) {
                annotationAttributes.forEach((name, value) -> System.out.printf("Annotation @%s attribute %s = %s\n",
                        ClassUtils.getShortName(metaAnnotation), name, value));
            }
        });
    }
}
```

运行程序，可以得到：
```txt
Annotation @Transactional attribute isolation = DEFAULT
Annotation @Transactional attribute noRollbackFor = [Ljava.lang.Class;@4d591d15
Annotation @Transactional attribute noRollbackForClassName = [Ljava.lang.String;@65ae6ba4
Annotation @Transactional attribute propagation = REQUIRED
Annotation @Transactional attribute readOnly = false
Annotation @Transactional attribute rollbackFor = [Ljava.lang.Class;@48cf768c
Annotation @Transactional attribute rollbackForClassName = [Ljava.lang.String;@65ae6ba4
Annotation @Transactional attribute timeout = -1
Annotation @Transactional attribute transactionManager = 
Annotation @Transactional attribute value = 
Annotation @Service attribute value = 
Annotation @Component attribute value = 
```

可以看出，使用`AnnotationMetadata` API重构后的代码降低了递归查找元注解的逻辑复杂度。

根据[`AnnotationMetadata`的JavaDoc](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/core/type/AnnotationMetadata.html)，可以看到其实现有`AnnotationMetadataReadingVisitor`和`StandardAnnotationMetadata`，二者实现方式不同，语义相同。

基于`ASM`实现的更适用于指定Java package扫描Spring模式注解，因为反射实现时，那么被反射的类必须要被`ClassLoader`加载。使用`ASM`方式无需全部将类加载，若当Class已经被加载并能被程序获取时，则直接使用反射即可。

`ASM`和反射性能比较：
```java
package deep.in.springframework;

import deep.in.springframework.annotation.TransactionalService;
import org.springframework.core.type.AnnotationMetadata;
import org.springframework.core.type.classreading.MetadataReader;
import org.springframework.core.type.classreading.SimpleMetadataReaderFactory;

public class AnnotationMetadataPerformanceBootstrap {

    public static void main(String[] args) throws Exception {
        AnnotationMetadata byReflection = AnnotationMetadata.introspect(TransactionalService.class);

        SimpleMetadataReaderFactory factory = new SimpleMetadataReaderFactory();
        MetadataReader metadataReader = factory.getMetadataReader(TransactionalService.class.getName());
        AnnotationMetadata byASM = metadataReader.getAnnotationMetadata();

        int times = 10000 * 10;//100,000

        testAnnotationMetadataPerformance(byReflection, times);
        testAnnotationMetadataPerformance(byASM, times);

        times = 10000 * 100;//1,000,000

        testAnnotationMetadataPerformance(byReflection, times);
        testAnnotationMetadataPerformance(byASM, times);

        times = 10000 * 1000;//10,000,000

        testAnnotationMetadataPerformance(byReflection, times);
        testAnnotationMetadataPerformance(byASM, times);

        times = 10000 * 10000;//100,000,000

        testAnnotationMetadataPerformance(byReflection, times);
        testAnnotationMetadataPerformance(byASM, times);

    }

    private static void testAnnotationMetadataPerformance(AnnotationMetadata annotationMetadata, int times) {
        long startTime = System.currentTimeMillis();
        for (int i = 0; i < times; i++) {
            annotationMetadata.getAnnotationTypes();
        }
        long endTime = System.currentTimeMillis();
        System.out.printf("%d %s.getAnnotationTypes() took %s ms\n", times, annotationMetadata.getClass().getSimpleName(), endTime - startTime);
    }
}
```

运行程序，可以得到：
```txt
100000 StandardAnnotationMetadata.getAnnotationTypes() took 14 ms
100000 SimpleAnnotationMetadata.getAnnotationTypes() took 5 ms
1000000 StandardAnnotationMetadata.getAnnotationTypes() took 7 ms
1000000 SimpleAnnotationMetadata.getAnnotationTypes() took 1 ms
10000000 StandardAnnotationMetadata.getAnnotationTypes() took 7 ms
10000000 SimpleAnnotationMetadata.getAnnotationTypes() took 0 ms
100000000 StandardAnnotationMetadata.getAnnotationTypes() took 0 ms
100000000 SimpleAnnotationMetadata.getAnnotationTypes() took 0 ms
```

虽然执行速度已经很快(测试环境：`Intel(R) Core(TM) i9-9880H CPU @ 2.30GHz` \ `16G DDR4 2400MHz`)，但`SimpleAnnotationMetadata`的性能还是要优于`StandardAnnotationMetadata`

