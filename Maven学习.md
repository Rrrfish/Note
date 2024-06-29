# Maven

## Maven坐标

**type**:  依赖项的类型，即依赖项的坐标中的packaging元素。它决定了依赖项的格式和行为。常见的类型有jar、war、pom等。默认为`Jar`。

**scope**:  依赖项的作用范围，即依赖项在哪些类路径下可用。

**optional**:  标记依赖是否可选。

**exclusion**：用来排除传递性依赖（例如a依赖于b、c、d，如果想要引入a，需要再下载b、c、d）。



## Scope属性

scope属性有几个可选项。

**compile**：默认的依赖有效范围。此种依赖在编译、运行、测试时均有效。

**provided**：在编译、测试时有效。比如`Lombok`在编译时就已经转化为对应代码了，运行时不需要这个依赖。

**runtime**：在测试、运行时有效。以 JDBC 驱动为例，通常代码只依赖于 `javax.sql` 和 `java.sql` 这些由 JDK 提供的接口来编程。**JDK 提供了 JDBC 的接口**，这些接口定义了访问数据库所需的各种方法，但是**具体的数据库驱动则由各个数据库厂商实现并提供**。因此，JDBC 驱动的依赖设置为 `runtime`，这样它们在编译时不会被包含在内，但在运行时会被加入到运行环境中来提供具体驱动。

**test**：只在测试时有效，如`JUnit`。

**system**：作用域和provided一样，但是是从本地导入Jar包。

```xml
<dependency>
	<groupId>javax.jntm</groupId>
    <artifactId>yyds</artifactId>
    <version>1.3.5</version>
    <scope>system</scope>
    <systemPath>D://haha/test.jar</systemPath>
</dependency>
```



## 依赖排除（Dependency Exclusions）

### 使用情形一

想要添加`project-b`的依赖，但是`project-b`自身`pom.xml`文件引入的`project-c`依赖版本太低，我们想要更高版本的`project-c`。

示例

```xml
<dependencies>
    <dependency>
        <groupId>org.example</groupId>
        <artifactId>project-b</artifactId>
        <version>1.0.0</version>
        <exclusions>
            <exclusion>
                <groupId>com.domain</groupId>
                <artifactId>project-c</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    <dependency>
        <groupId>com.domain</groupId>
        <artifactId>project-c</artifactId>
        <version>4.0.0</version>
    </dependency>
</dependencies>
```

这个示例可以做到添加依赖`project-b`的同时不添加`project-b`所依赖的低版本（如1.0）`project-c`，而是手动加入依赖4.0版本的`project-c`。

其实**直接声明**对project-c的依赖，根据Maven的**最近优先原则（nearest-wins strategy）**，就会直接依赖project-c的4.0.0版本，不使用`exclusion`也可以做到同样的效果。但是使用了exclusion标签，其语义上会更明确。



### 使用情形二

`project-a`包含众多功能，但我们只想使用其中部分功能，如果`project-a`的强大功能是因为加载了众多依赖实现的，那么我们可以对其部分冗余依赖进行排除，只保留我们需要的部分即可。



## Maven依赖的最近优先原则（nearest-wins strategy）

当 Maven 构建项目时，它会创建一个依赖树，这是一个层次化的结构，显示了项目依赖的所有库及其子依赖。最近优先原则的工作方式是：

1. **构建依赖树**：Maven 首先解析项目的直接依赖，并递归地解析这些依赖的依赖，最终形成一个完整的依赖树。
2. **应用最近优先原则**：当同一个库的不同版本在依赖树中多次出现时，Maven 会选择距离项目“最近”的版本。所谓“最近”，是指这个版本在依赖树中的层级最低，也就是从项目到该依赖版本的路径中间经过的节点最少。
3. **决定依赖版本**：基于这个原则，如果一个库在依赖树中多次出现，靠近根的版本（即直接依赖或间接依赖层级较浅的版本）会被使用，而更深层级的同一库的版本会被忽略。

### 示例

假设有以下依赖关系：

- 项目 A 依赖于库 B（版本1.0）和库 C（版本2.0）。
- 库 B（版本1.0）依赖于库 D（版本3.0）。
- 库 C（版本2.0）依赖于库 D（版本3.1）。

依赖树可能如下：
```mathematica
A
├── B v1.0
│   └── D v3.0
└── C v2.0
    └── D v3.1
```

根据最近优先原则，库 D 的版本选择将会是 D v3.0，因为它在依赖树中的位置比 D v3.1 更接近项目 A。

### 限制和考量

- **不是总是最优解**：最近优先原则简化了依赖管理，但并不总是得到最合适的结果。在某些情况下，项目可能需要更高版本的库以修复安全问题或兼容性问题。
- **可以通过依赖管理进行干预**：如果默认的决策不合适，可以在 Maven 的 `pom.xml` 文件中通过明确指定依赖版本或使用 `<dependencyManagement>` 标签来覆盖默认行为。





## 可选依赖 （Optional Dependency）

可选依赖允许库的开发者声明某些依赖是非必需的，使用者可以根据自己的需要选择是否引入这些依赖。





## 依赖继承 （Dependency inheritance）

为了项目的正确运行，必须让所有的子模块使用依赖项的统一版本，必须确保应用的各个项目的依赖项和版本一致，才能保证测试的和发布的是相同的结果。

### `<dependencies>`和`<dependencyManagement>`

- Artifacts specified in the **`<dependencies>`** section will ALWAYS be included as a dependency of the child module(s).
- Artifacts specified in the **`<dependencyManagement>`** section, will only be included in the child module if they were also specified in the **`<dependencies>`** section of the child module itself. 

（节选自`stackOverflow`）

如果双亲项目中使用了`<dependencyManagement>`引入某依赖，子项目在`<dependencies>`中引入该依赖时不需要声明版本。





## Maven常用命令

`clean`:  清理整个target文件夹，可以解决某些`SpringBoot`缓存未更新的问题。

`validate`: 验证项目可用性。

`compile`：将项目编译成`.class`文件。

`install`：打包的工件安装到本地 Maven 存储库`~/.m2/repository`中，使其可用于同一计算机上的其他项目。

`verify`：按照顺序执行每一个默认生命周期阶段。该命令用于验证项目是否符合质量标准和各种规则。它通常在构建周期的较后阶段执行，通常在verify阶段执行。在这个阶段，插件可能会运行各种验证，例如执行集成测试、检查代码质量、运行静态分析等。

`package`：编译后的代码打包成 JAR 或 WAR 文件，具体取决于项目类型。



## `test`命令

- 只会执行Maven项目中src/test/java目录下的测试类。
- 只有类的命名规范满足XxxTest.java才会执行。