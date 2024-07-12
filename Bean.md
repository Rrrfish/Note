# Bean的注册与配置

我们在`xml`文件中注册Bean。

```xml
// src/main/java/resource/application.xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean/>
</beans>
```

在Main函数中，我们需要实现`ApplicationContext`类，这里由于我们使用`xml`文件进行Bean的管理，所以实现类使用`ClassPathXmlApplicationContext`。

```java
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Main {
    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
    }
}
```





## ClassPath 类路径

在 Spring 框架中，`ClassPathXmlApplicationContext` 是一个常用的 ApplicationContext 实现，它从**类路径（ClassPath）**中加载配置文件。类路径是 Java 应用的一部分，**JVM**在类路径中查找加载类和其他资源（如配置文件）。



## 注册Bean

```java
// src/main/java/resource/application.xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean class="com.example.entity.CsStudent" name="csStudent"/>
</beans>
```

可以用`name`来指定Bean的名字，在`ApplicationContext`中获取Bean时可以用`name`也可以用`Class`。

```java
// src/main/java/com/example/entity/CsStudent
@Data
public class CsStudent {
    String name;

    public void hello() {
        System.out.println("计算机学生出生第一句话：Hello World!");
    }
}

// src/main/java/com/example/Main
public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
//        CsStudent csStudent = (CsStudent) context.getBean(CsStudent.class);
        CsStudent csStudent = (CsStudent) context.getBean("csStudent");
        System.out.println(csStudent);
        csStudent.hello();
    }
```

```shell
CsStudent(name=null)
计算机学生出生第一句话：Hello World!
```

`getBean(CsStuddent.class)`和`getBean("csStudent")`都一样，但是后者在注册Bean时需要指明。



## Bean获取、接口与实现类

```java
// src/main/java/com/example/entity/Student
import lombok.Data;
@Data
public abstract class Student {
    String name;
}

// src/main/java/com/example/entity/CsStudent
@Data
public class CsStudent {
    String name;

    public void hello() {
        System.out.println("计算机学生出生第一句话：Hello World!");
    }
}
```

延伸一下这里学到的一个`Lombok`的知识，注意到此时`CsStudent`的`@Data`爆黄。

### `Lombok`的`@Data`注解

```
Generating equals/hashCode implementation but without a call to superclass, even though this class does not extend java.lang.Object. If this is intentional, add '(callSuper=false)' to your type.
```

Lombok的`@Data`注解为类自动生成以下内容：

- 所有字段的Getter和Setter
- `toString()`方法
- `equals()`和`hashCode()`方法
- 一个全参构造器

这条警告是Lombok在提醒，在生成`equals`和`hashCode`方法时没有调用超类（superclass）的相应方法。在Java中，所有的类默认继承自`java.lang.Object`，除非显式地指定其他父类。如果你的类`CsStudent`声明它扩展了`Student`，那么这里的超类是`Student`。

如果类`CsStudent`继承了`Student`，而`Student`类包含了一些字段，那么为了正确地处理这些字段在`equals`和`hashCode`方法中的逻辑，程序员应该告诉`Lombok`也调用超类的方法，或者明确指定不调用。

#### 修改方法

在`@Data`注解中使用`callSuper=true`来确保`equals`和`hashCode`方法也考虑到超类的字段。

```java
@Data(callSuper=true) // 告诉Lombok在生成的方法中调用超类的方法
public class CsStudent extends Student {
    // --snip--
}
```





























