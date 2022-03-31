# 概述

## 1. 什么是Junit 5

Junit 5 由三个子模块组成：

**JUnit 5 = *JUnit Platform* + *JUnit Jupiter* + *JUnit Vintage***

- Junit Platform是整个测试框架的基础。它提供了命令行，IDE和构建工具等方式执行测试的支持。

- Junit Jupiter包含了Junit 5中用来编写测试代码和扩展代码时新的编程模型和扩展模型。

- Junit Vintage提供了运行基于Junit 3和Junit 4的测试环境。

## 2. 支持的Java版本

Junit 5运行时需要Java 8（或更高）版本。但是编译时可以使用低版本的JDK。

# 编写测试

## 1. 测试类和测试方法

- 测试类：任何的顶级类，静态类，或是使用@Nested修饰的包含至少一个测试方法的类都可以时测试类。**测试类不能是抽象的，并且必须有一个无参构造方法。**

- 测试方法：任何被@Test，@RepeatedTest，@ParameterizedTest，@TestFactory，或@TestTemplate直接注解或元注解的实例方法都是测试方法。

- 生命周期方法：任何被@BeforeAll，@AfterAll，@BeforeEach，或@AfterEach直接注解或元注解的方法都是生命周期方法。
  
  **测试方法和生命周期方法可以在当前类中声明，也可以从超类继承或是从接口继承。另外，它们不能是抽象的，也不能有返回值（@TestFactory注解的方法除外）。**

## 2. Display Names（显示名称）

使用此注解可以在测试报告和IDE中给测试类和测试方法自定义显示名称。该显示名称可以使用空白，特殊字符，甚至是emoji等。

```java
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;

@DisplayName("A special test case")
class DisplayNameDemo {

    @Test
    @DisplayName("Custom test name containing spaces")
    void testWithDisplayNameContainingSpaces() {
    }

    @Test
    @DisplayName("╯°□°）╯")
    void testWithDisplayNameContainingSpecialCharacters() {
    }

    @Test
    @DisplayName("😱")
    void testWithDisplayNameContainingEmoji() {
    }

}
```

### 2.1 自定义生成器

显示名称可以按照一定的规则自动生成。使用@DisplayNameGeneration注解可以自定义显示名称的生成。以下表格展示了Junit 5中可以使用的生成器：

| 生成器                 | 说明                         |
|:------------------- | -------------------------- |
| Standard            | Junit 5中标准的生成器行为。          |
| Simple              | 显示名称为方法名（不带括号）。            |
| ReplaceUnderscores  | 将测试方法中的下划线替换成空格作为显示名称。     |
| IndicativeSentences | 将测试类和测试方法名通过分隔符连接起来作为显示名称。 |

以下是例子：

```java
package displayname;

import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.DisplayNameGeneration;
import org.junit.jupiter.api.DisplayNameGenerator;
import org.junit.jupiter.api.IndicativeSentencesGeneration;
import org.junit.jupiter.api.Nested;
import org.junit.jupiter.api.Test;

public class DisplayNameGeneratorDemo {

    @Nested
    @DisplayNameGeneration(DisplayNameGenerator.Standard.class)
    class Standard {

        @Test
        void this_is_test_1() {
            // Standard生成器不会自动生成显示名称。
        } 

        @Test
        @DisplayName("Dispaly Name takes precedence over display names generated.")
        void this_is_test_2() {
            // 使用@DisplayName注解优先于使用@DisplayNameGeneration注解。
        }
    }

    @Nested
    @DisplayNameGeneration(DisplayNameGenerator.Simple.class)
    class Simple {

        @Test
        void this_is_test_1() {
            // 显示名称为：this_is_test_1
        } 
    }

    @Nested
    @DisplayNameGeneration(DisplayNameGenerator.ReplaceUnderscores.class)
    class ReplaceUnderscores {

        @Test
        void this_is_test_1() {
            // 显示名称为：this is test 1
        } 
    }

    @Nested
    @IndicativeSentencesGeneration(separator = " -> ", generator = DisplayNameGenerator.ReplaceUnderscores.class)
    class IndicativeSentences {

        @Test
        void this_is_test_1() {
            // 显示名称为：IndicativeSentences -> this is test 1
            // 方法名不带下划线是因为使用了ReplaceUnderscores生成器。
        } 
    }
}
```

### 2.2 设置默认的生成器

如果要将`ReplaceUnderscore`生成器设置为默认生成器，可以在`src/test/resource/junit-platform.properties`中定义如下属性：

```properties
junit.jupiter.displayname.generator.default = org.junit.jupiter.api.DisplayNameGenerator$ReplaceUnderscores
```

另外，显示名称的优先规则如下：

1. 使用@DisplayName注解

2. 使用@DisplayNameGeneration注解

3. 默认配置的生成器

4. 调用`org.junit.jupiter.api.DisplayNameGenerator.Standard`

## 3. Assertions（断言）

**在Junit 5中所有的断言都是org.junit.jupiter.api.Assertions包中的静态方法。**

```java
import static java.time.Duration.ofMillis;
import static java.time.Duration.ofMinutes;
import static org.junit.jupiter.api.Assertions.assertAll;
import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertNotNull;
import static org.junit.jupiter.api.Assertions.assertThrows;
import static org.junit.jupiter.api.Assertions.assertTimeout;
import static org.junit.jupiter.api.Assertions.assertTimeoutPreemptively;
import static org.junit.jupiter.api.Assertions.assertTrue;

import java.util.concurrent.CountDownLatch;

import example.domain.Person;
import example.util.Calculator;

import org.junit.jupiter.api.Test;

class AssertionsDemo {

    private final Calculator calculator = new Calculator();

    private final Person person = new Person("Jane", "Doe");

    @Test
    void standardAssertions() {
        assertEquals(2, calculator.add(1, 1));
        assertEquals(4, calculator.multiply(2, 2),
                "The optional failure message is now the last parameter");
        assertTrue('a' < 'b', () -> "Assertion messages can be lazily evaluated -- "
                + "to avoid constructing complex messages unnecessarily.");
    }

    @Test
    void groupedAssertions() {
        // 在一个分组的断言中，所有的断言都会被执行，所有的失败都会一起报告。 
        assertAll("person",
            () -> assertEquals("Jane", person.getFirstName()),
            () -> assertEquals("Doe", person.getLastName())
        );
    }

    @Test
    void dependentAssertions() {
        // 在代码块内，如果断言失败，则将跳过同一块中的后续代码。
        assertAll("properties",
            () -> {
                String firstName = person.getFirstName();
                assertNotNull(firstName);

                // 仅当前一个断言有效时才执行。
                assertAll("first name",
                    () -> assertTrue(firstName.startsWith("J")),
                    () -> assertTrue(firstName.endsWith("e"))
                );
            },
            () -> {
                // 在代码块内，如果断言失败，则将跳过同一块中的后续代码。
                String lastName = person.getLastName();
                assertNotNull(lastName);

                // 仅当前一个断言有效时才执行。
                assertAll("last name",
                    () -> assertTrue(lastName.startsWith("D")),
                    () -> assertTrue(lastName.endsWith("e"))
                );
            }
        );
    }

    @Test
    void exceptionTesting() {
        Exception exception = assertThrows(ArithmeticException.class, () ->
            calculator.divide(1, 0));
        assertEquals("/ by zero", exception.getMessage());
    }

    @Test
    void timeoutNotExceeded() {
        // 执行两分钟以内的任务
        assertTimeout(ofMinutes(2), () -> {
            // 执行任务不超过两分钟
        });
    }

    @Test
    void timeoutNotExceededWithResult() {
        // 断言成功后返回对象
        String actualResult = assertTimeout(ofMinutes(2), () -> {
            return "a result";
        });
        assertEquals("a result", actualResult);
    }

    @Test
    void timeoutExceededWithPreemptiveTermination() {
        assertTimeoutPreemptively(ofMillis(10), () -> {
            new CountDownLatch(1).await();
        });
    }
}
```

**注意**

assertTimeoutPreemptively()属于抢占式超时，与声明式超时相反，assertTimeoutPreemptively()将另起一个新的线程执行任务。因此如果使用`java.lang.ThreadLocal`存储可能会有副作用。

## 4. Assumptions（假设）

Assumptions方法中可以使用lambda表达式和方法引用。

**在Junit 5中所有的假设方法都是org.junit.jupiter.api.Assumptions包中的静态方法。**

```java
import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assumptions.assumeTrue;
import static org.junit.jupiter.api.Assumptions.assumingThat;

import example.util.Calculator;

import org.junit.jupiter.api.Test;

class AssumptionsDemo {

    private final Calculator calculator = new Calculator();

    @Test
    void testOnlyOnCiServer() {
        assumeTrue("CI".equals(System.getenv("ENV")));
    }

    @Test
    void testOnlyOnDeveloperWorkstation() {
        // 使用lambda表达式
        assumeTrue("DEV".equals(System.getenv("ENV")),
            () -> "Aborting test: not on developer workstation");
    }
}
```

## 5. Disabling Tests（禁用测试）

在测试类上使用@Disabled注解可以禁用整个测试类。

在测试方法上使用@Disabled注解可以禁用单独的一个测试方法。

## 6. Conditional Test Execution（基于条件执行测试）

`org.junit.jupiter.api.condition`包提供了基于注解的条件使开发人员可以以编程的方式基于特定条件启用或禁用测试。当使用了多个条件时，一旦其中的一个条件返回disabled，则这个测试将会被禁用。

### 6.1 Operating System Conditions（操作系统条件）

可以通过 @EnabledOnOs 和 @DisabledOnOs 注解在特定操作系统上启用或禁用测试。

### 6.2 Java Runtime Environment Conditions

可以通过 @EnabledOnJre 和 @DisabledOnJre 注解在特定版本的 Java 运行时环境 (JRE) 上启用或禁用测试，或者通过 @EnabledForJreRange 和 @DisabledForJreRange 注解在特定范围的 JRE 版本上启用或禁用测试。

### 6.3 System Property Conditions

可以通过`@EnabledIfSystemProperty`和`@DisabledIfSystemProperty`注解根据 JVM 系统属性的值启用或禁用测试。通过matches属性提供的值将被解释为正则表达式。

```java
@Test
@EnabledIfSystemProperty(named = "os.arch", matches = ".*64.*")
void onlyOn64BitArchitectures() {
    // ...
}

@Test
@DisabledIfSystemProperty(named = "ci-server", matches = "true")
void notOnCiServer() {
    // ...
}
```

### 6.4 Environment Variable Conditions

可以通过`@EnabledIfEnvironmentVariable`和`@DisabledIfEnvironmentVariable`注解基于来自底层操作系统的命名环境变量的值启用或禁用测试。通过matches属性提供的值将被解释为正则表达式。

```java
@Test
@EnabledIfEnvironmentVariable(named = "ENV", matches = "staging-server")
void onlyOnStagingServer() {
    // ...
}

@Test
@DisabledIfEnvironmentVariable(named = "ENV", matches = ".*development.*")
void notOnDeveloperWorkstation() {
    // ...
}
```

### 6.5 Custom Conditions

可以基于通过`@EnabledIf`和`@DisabledIf`注解的方法的布尔返回值来启用或禁用测试。该方法通过其名称提供给注解。

```java
@Test
@EnabledIf("customCondition")
void enabled() {
    // ...
}

@Test
@DisabledIf("customCondition")
void disabled() {
    // ...
}

boolean customCondition() {
    return true;
}
```

条件方法可以位于测试类之外。在这种情况下，必须通过其完全限定名来引用它。

```java
package example;

import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.condition.EnabledIf;

class ExternalCustomConditionDemo {

    @Test
    @EnabledIf("example.ExternalCondition#customCondition")
    void enabled() {
        // ...
    }
}

class ExternalCondition {

    static boolean customCondition() {
        return true;
    }
}
```

**在条件方法在测试类外时，条件方法必须是静态的。**

## 7. Tagging and Filtering（标记和过滤）

测试类和方法可以通过@Tag注解进行标记。这些标记稍后可以用于过滤。

## 8. Test Execution Order（测试执行顺序）

默认情况下，测试类和测试方法将使用确定性但故意不明显的算法进行排序。也就是多次执行时的顺序一致但是以什么样的顺序执行是没有规律的。

### 8.1 Method Order（测试方法的执行顺序）

要控制测试方法的执行顺序，可以使用@TestMethodOrder注解测试类或测试接口，并指定所需的MethodOrderer实现。

| Method Orderer                | 排序规则                                    |
| ----------------------------- | --------------------------------------- |
| MethodOrderer.DisplayName     | 根据显示名称按字母数字对测试方法排序                      |
| MethodOrderer.Methodname      | 根据名称和形参列表按字母数字对测试方法进行排序。**不推荐使用，以后将删除** |
| MethodOrderer.OrderAnnotation | 根据通过@Order注解指定的值对测试方法进行数字排序             |
| MethodOrderer.Random          | 对测试方法进行伪随机排序并支持自定义种子的配置                 |
| MethodOrderer.Alphanumeric    | 根据名称和形式参数列表按字母数字对测试方法进行排序。              |

以下示例展示了如何保证测试方法按照@Order注解指定的顺序执行：

```java
import org.junit.jupiter.api.MethodOrderer.OrderAnnotation;
import org.junit.jupiter.api.Order;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.TestMethodOrder;

@TestMethodOrder(OrderAnnotation.class)
class OrderedTestsDemo {

    @Test
    @Order(1)
    void nullValues() {
        // perform assertions against null values
    }

    @Test
    @Order(2)
    void emptyValues() {
        // perform assertions against empty values
    }

    @Test
    @Order(3)
    void validValues() {
        // perform assertions against valid values
    }

}
```

### 8.2 设置默认的测试方法执行顺序

```properties
junit.jupiter.testmethod.order.default = org.junit.jupiter.api.MethodOrderer$OrderAnnotation
```

### 8.3 Class Order（测试类的执行顺序）

要控制测试类的执行顺序，可以使用@TestClassOrder注解测试类，并指定所需的ClassOrderer实现。

| Class Order                  | 排序规则                       |
| ---------------------------- | -------------------------- |
| ClassOrderer.ClassName       | 根据完全限定的类名按字母数字对测试类进行排序     |
| ClassOrderer.DisplayName     | 根据显示名称按字母数字对测试类进行排序        |
| ClassOrderer.OrderAnnotation | 根据通过@Order注解指定的值对测试类进行数字排序 |
| ClassOrderer.Random          | 对测试类进行伪随机排序并支持自定义种子的配置     |

以下示例展示了如何保证测试类按照@Order注解指定的顺序执行：

```java
import org.junit.jupiter.api.ClassOrderer;
import org.junit.jupiter.api.Nested;
import org.junit.jupiter.api.Order;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.TestClassOrder;

@TestClassOrder(ClassOrderer.OrderAnnotation.class)
class OrderedNestedTestClassesDemo {

    @Nested
    @Order(1)
    class PrimaryTests {

        @Test
        void test1() {
        }
    }

    @Nested
    @Order(2)
    class SecondaryTests {

        @Test
        void test2() {
        }
    }
}
```

### 8.4 设置默认的测试类执行顺序

```properties
junit.jupiter.testclass.order.default = org.junit.jupiter.api.ClassOrderer$OrderAnnotation
```

## 9. Test Instance Lifecycle


