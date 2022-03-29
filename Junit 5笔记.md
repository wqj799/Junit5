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

## 2.Display Names（显示名称）

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

### 2.1自定义生成器

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

### 2.2设置默认的生成器

如果要将`ReplaceUnderscore`生成器设置为默认生成器，可以在`src/test/resource/junit-platform.properties`中定义如下属性：

```properties
junit.jupiter.displayname.generator.default = org.junit.jupiter.api.DisplayNameGenerator$ReplaceUnderscores
```

另外，显示名称的优先规则如下：

1. 使用@DisplayName注解

2. 使用@DisplayNameGeneration注解

3. 默认配置的生成器

4. 调用`org.junit.jupiter.api.DisplayNameGenerator.Standard`

## 3.Assertions（断言）

**在Junit 5中所有的断言都是org.junit.jupiter.api.Assertions类中的静态方法。**

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


