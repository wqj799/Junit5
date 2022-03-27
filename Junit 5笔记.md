# 概述

## 1.什么是Junit 5

Junit 5 由三个子模块组成：

**JUnit 5 = *JUnit Platform* + *JUnit Jupiter* + *JUnit Vintage***

- Junit Platform是整个测试框架的基础。它提供了命令行，IDE和构建工具等方式执行测试的支持。

- Junit Jupiter包含了Junit 5中用来编写测试代码和扩展代码时新的编程模型和扩展模型。

- Junit Vintage提供了运行基于Junit 3和Junit 4的测试环境。

## 2.支持的Java版本

Junit 5运行时需要Java 8（或更高）版本。但是编译时可以使用低版本的JDK。

# 编写测试

## 1.测试类和测试方法

- 测试类：任何的顶级类，静态类，或是使用@Nested修饰的包含至少一个测试方法的类都可以时测试类。**测试类不能是抽象的，并且必须有一个无参构造方法。**

- 测试方法：任何被@Test，@RepeatedTest，@ParameterizedTest，@TestFactory，或@TestTemplate直接注解或元注解的实例方法都是测试方法。

- 生命周期方法：任何被@BeforeAll，@AfterAll，@BeforeEach，或@AfterEach直接注解或元注解的方法都是生命周期方法。
  
  **测试方法和生命周期方法可以在当前类中声明，也可以从超类继承或是从接口继承。另外，它们不能是抽象的，也不能有返回值（@TestFactory注解的方法除外）。**


