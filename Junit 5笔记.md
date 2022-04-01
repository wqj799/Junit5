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

## 9. Test Instance Lifecycle（测试实例生命周期)

- 在测试类上使用@TestInstance(Lifecycle.PER_METHOD)注解，Junit在执行每个测试方法时都将创建一个新的实例。这是Junit的默认行为。

- 在测试类上使用@TestInstance(Lifecycle.PER_CLASS)注解，Junit在同一个实例中执行所有的测试方法。使用此模式可以在非静态方法以及接口默认方法上声明@BeforeAll和@AfterAll。因此还可以在@Nested注解的测试类中使用@BeforeAll和@AfterAll。

### 9.1 更改默认的测试实例生命周期

```properties
junit.jupiter.testinstance.lifecycle.default = per_class
```

## 10. Nested Tests（嵌套测试）

利用嵌套测试可以表达几组测试之间的关系。如下实例所示：

```java
@DisplayName("A stack")
class TestingAStackDemo {

    Stack<Object> stack;

    @Test
    @DisplayName("is instantiated with new Stack()")
    void isInstantiatedWithNew() {
        new Stack<>();
    }

    @Nested
    @DisplayName("when new")
    class WhenNew {

        @BeforeEach
        void createNewStack() {
            stack = new Stack<>();
        }

        @Test
        @DisplayName("is empty")
        void isEmpty() {
            assertTrue(stack.isEmpty());
        }

        @Test
        @DisplayName("throws EmptyStackException when popped")
        void throwsExceptionWhenPopped() {
            assertThrows(EmptyStackException.class, stack::pop);
        }

        @Test
        @DisplayName("throws EmptyStackException when peeked")
        void throwsExceptionWhenPeeked() {
            assertThrows(EmptyStackException.class, stack::peek);
        }

        @Nested
        @DisplayName("after pushing an element")
        class AfterPushing {

            String anElement = "an element";

            @BeforeEach
            void pushAnElement() {
                stack.push(anElement);
            }

            @Test
            @DisplayName("it is no longer empty")
            void isNotEmpty() {
                assertFalse(stack.isEmpty());
            }

            @Test
            @DisplayName("returns the element when popped and is empty")
            void returnElementWhenPopped() {
                assertEquals(anElement, stack.pop());
                assertTrue(stack.isEmpty());
            }

            @Test
            @DisplayName("returns the element when peeked but remains not empty")
            void returnElementWhenPeeked() {
                assertEquals(anElement, stack.peek());
                assertFalse(stack.isEmpty());
            }
        }
    }
}
```

**只有非静态测试类（即内部类）可以用作@Nested测试类。嵌套类支持完整的生命周期，但是@BeforeAll和@AfterAll注解默认不起作用，原因是Java不允许内部类中的静态成员，而这两个注解必须是静态方法。。**

## 11. Dependency Injection for Constructors and Methods（构造函数和方法的依赖注入）

Junit 5允许测试构造函数和方法具有参数。参数的类型需要能被`ParameterResolver`解析器解析。

Junit 5支持以下三种解析器：

1. TestInfoParameterResolver：如果构造函数或方法的参数是TestInfo类型,则TestInfoParameterResolver解析器将提供TestInfo实例。TestInfo可以获取当前测试的信息，如DisplayName，Tags等。
   
   ```java
   @DisplayName("TestInfo Demo")
   class TestInfoDemo {
   
       TestInfoDemo(TestInfo testInfo) {
           assertEquals("TestInfo Demo", testInfo.getDisplayName());
       }
   
       @BeforeEach
       void init(TestInfo testInfo) {
           String displayName = testInfo.getDisplayName();
           assertTrue(displayName.equals("TEST 1") || displayName.equals("test2()"));
       }
   
       @Test
       @DisplayName("TEST 1")
       @Tag("my-tag")
       void test1(TestInfo testInfo) {
           assertEquals("TEST 1", testInfo.getDisplayName());
           assertTrue(testInfo.getTags().contains("my-tag"));
       }
   
       @Test
       void test2() {
       }
   
   }
   ```

2. RepetitionInfoParameterResolver：如果@RepeatedTest、@BeforeEach 或@AfterEach 方法中的方法参数是RepetitionInfo类型，则RepetitionInfoParameterResolver将提供RepetitionInfo的实例。RepetitionInfo用于获取当前测试的重复信息和重复总数。

3. TestReporterParameterResolver：如果构造函数或方法参数的类型为 TestReporter，则TestReporterParameterResolver将提供TestReporter的实例。TestReporter用于获取当前测试运行的附加数据。
   
   ```java
   class TestReporterDemo {
   
       @Test
       void reportSingleValue(TestReporter testReporter) {
           testReporter.publishEntry("a status message");
       }
   
       @Test
       void reportKeyValuePair(TestReporter testReporter) {
           testReporter.publishEntry("a key", "a value");
       }
   
       @Test
       void reportMultipleKeyValuePairs(TestReporter testReporter) {
           Map<String, String> values = new HashMap<>();
           values.put("user name", "dk38");
           values.put("award year", "1974");
   
           testReporter.publishEntry(values);
       }
   
   }
   ```

## 12. Repeated Tests（重复测试）

使用@RepeatedTest注解并指定所需的重复总数，提供了将测试重复指定次数的能力。

可以通过@RepeatedTest注解的name属性为每个重复配置自定义显示名称。name属性支持以下占位符：

- {displayName}：@RepeatedTest 方法的显示名称

- {currentRepetition}：当前重复次数

- {totalRepetitions}：总重复次数

下面是一些例子：

```java
class RepeatedTestsDemo {

    private Logger logger = // ...

    @BeforeEach
    void beforeEach(TestInfo testInfo, RepetitionInfo repetitionInfo) {
        int currentRepetition = repetitionInfo.getCurrentRepetition();
        int totalRepetitions = repetitionInfo.getTotalRepetitions();
        String methodName = testInfo.getTestMethod().get().getName();
        logger.info(String.format("About to execute repetition %d of %d for %s", //
            currentRepetition, totalRepetitions, methodName));
    }

    @RepeatedTest(10)
    void repeatedTest() {
        // ...
    }

    @RepeatedTest(5)
    void repeatedTestWithRepetitionInfo(RepetitionInfo repetitionInfo) {
        assertEquals(5, repetitionInfo.getTotalRepetitions());
    }

    @RepeatedTest(value = 1, name = "{displayName} {currentRepetition}/{totalRepetitions}")
    @DisplayName("Repeat!")
    void customDisplayName(TestInfo testInfo) {
        assertEquals("Repeat! 1/1", testInfo.getDisplayName());
    }

    @RepeatedTest(value = 1, name = RepeatedTest.LONG_DISPLAY_NAME)
    @DisplayName("Details...")
    void customDisplayNameWithLongPattern(TestInfo testInfo) {
        assertEquals("Details... :: repetition 1 of 1", testInfo.getDisplayName());
    }

    @RepeatedTest(value = 5, name = "Wiederholung {currentRepetition} von {totalRepetitions}")
    void repeatedTestInGerman() {
        // ...
    }

}
```

## 13. Parameterized Tests（参数化测试）

使用不同的参数多次运行测试。需要使用@ParameterizedTest注解，并声明一个数据源。

**启用参数化测试需要添加依赖：junit-jupiter-params**

### 13.1 Sources of Arguments（参数来源）

#### 13.1.1 @ValueSource

指定一个数组作为数据源。支持的类型如下：short，byte，int，long，float，double，char，boolean，java.lang.String，java.lang.Class。

```java
@ParameterizedTest
@ValueSource(ints = { 1, 2, 3 })
void testWithValueSource(int argument) {
    assertTrue(argument > 0 && argument < 4);
}
```

需要传入null值时，使用@NullSource注解

需要传入empty值时，使用@EmptySource注解

两者都需要时，使用@NullAndEmptySource注解

```java
@ParameterizedTest
@NullSource
@EmptySource
@ValueSource(strings = { " ", "   ", "\t", "\n" })
void nullEmptyAndBlankStrings(String text) {
    assertTrue(text == null || text.trim().isEmpty());
}

@ParameterizedTest
@NullAndEmptySource
@ValueSource(strings = { " ", "   ", "\t", "\n" })
void nullEmptyAndBlankStrings(String text) {
    assertTrue(text == null || text.trim().isEmpty());
}
```

#### 13.1.2 @EnumSource

使用枚举常量作为数据源。

```java
@ParameterizedTest
@EnumSource(ChronoUnit.class)
void testWithEnumSource(TemporalUnit unit) {
    assertNotNull(unit);
}
```

该注解的value属性可以省略，如果省略，则使用方法的第一个参数作为枚举类

```java
@ParameterizedTest
@EnumSource
void testWithEnumSourceWithAutoDetection(ChronoUnit unit) {
    assertNotNull(unit);
}
```

该注解的name属性可以指定使用枚举中的哪些常量，如果省略则使用全部的常量。

```java
@ParameterizedTest
@EnumSource(names = { "DAYS", "HOURS" })
void testWithEnumSourceInclude(ChronoUnit unit) {
    assertTrue(EnumSet.of(ChronoUnit.DAYS, ChronoUnit.HOURS).contains(unit));
}
```

该注解的mode属性可以对常量进行细粒度的控制。

```java
@ParameterizedTest
@EnumSource(mode = EXCLUDE, names = { "ERAS", "FOREVER" })
void testWithEnumSourceExclude(ChronoUnit unit) {
    assertFalse(EnumSet.of(ChronoUnit.ERAS, ChronoUnit.FOREVER).contains(unit));
}

@ParameterizedTest
@EnumSource(mode = MATCH_ALL, names = "^.*DAYS$")
void testWithEnumSourceRegex(ChronoUnit unit) {
    assertTrue(unit.name().endsWith("DAYS"));
}
```

#### 13.1.3 @MethodSource

可以引用测试类或外部类的一个或多个工厂方法。

- 测试类中的工厂方法必须是静态的，除非测试类使用@TestInstance(Lifecycle.PER_CLASS)注解。

- 外部类中的工厂方法必须始终是静态的。

- 这两类工厂方法不得接受任何参数。

- 每个工厂方法都必须返回一个参数流（即Stream<Arguments>）

```java
@ParameterizedTest
@MethodSource("stringProvider")
void testWithExplicitLocalMethodSource(String argument) {
    assertNotNull(argument);
}

static Stream<String> stringProvider() {
    return Stream.of("apple", "banana");
}
```

如果@MethodSource没有显式提供工厂方法名称，则将按照约定搜索与当前@ParameterizedTest 方法同名的工厂方法。

```java
@ParameterizedTest
@MethodSource
void testWithDefaultLocalMethodSource(String argument) {
    assertNotNull(argument);
}

static Stream<String> testWithDefaultLocalMethodSource() {
    return Stream.of("apple", "banana");
}
```

如果测试方法需要提供多个参数，那么工厂方法需要返回collection，stream，或Arguments实例数组或Object数组。

```java
@ParameterizedTest
@MethodSource("stringIntAndListProvider")
void testWithMultiArgMethodSource(String str, int num, List<String> list) {
    assertEquals(5, str.length());
    assertTrue(num >=1 && num <=2);
    assertEquals(2, list.size());
}

static Stream<Arguments> stringIntAndListProvider() {
    return Stream.of(
        arguments("apple", 1, Arrays.asList("a", "b")),
        arguments("lemon", 2, Arrays.asList("x", "y"))
    );
}
```

引用外部静态工厂方法需要提供其完全限定方法名。

```java
package example;

import java.util.stream.Stream;

import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.MethodSource;

class ExternalMethodSourceDemo {

    @ParameterizedTest
    @MethodSource("example.StringsProviders#tinyStrings")
    void testWithExternalMethodSource(String tinyString) {
        // test with tiny string
    }
}

class StringsProviders {

    static Stream<String> tinyStrings() {
        return Stream.of(".", "oo", "OOO");
    }
}
```

#### 13.1.4 @CsvSource

使用CSV格式的数据作为数据源。

```java
@ParameterizedTest
@CsvSource({
    "apple,         1",
    "banana,        2",
    "'lemon, lime', 0xF1",
    "strawberry,    700_000"
})
void testWithCsvSource(String fruit, int rank) {
    assertNotNull(fruit);
    assertNotEquals(0, rank);
}
```

默认使用逗号作为分隔符，单引号作为引号字符。

| Example Input                                                                           | Resulting Argument List       |
|:--------------------------------------------------------------------------------------- | ----------------------------- |
| `@CsvSource({ "apple, banana" })`                                                       | `"apple"`, `"banana"`         |
| `@CsvSource({ "apple, 'lemon, lime'" })`                                                | `"apple"`, `"lemon, lime"`    |
| `@CsvSource({ "apple, ''" })`                                                           | `"apple"`, `""`               |
| `@CsvSource({ "apple, " })`                                                             | `"apple"`, `null`             |
| `@CsvSource(value = { "apple, banana, NIL" }, nullValues = "NIL")`                      | `"apple"`, `"banana"`, `null` |
| `@CsvSource(value = { " apple , banana" }, ignoreLeadingAndTrailingWhitespace = false)` | `" apple "`, `" banana"`      |

#### 13.1.5 @CsvFileSource

使用来自类路径或本地文件系统的CSV文件作为数据源。

```java
@ParameterizedTest
@CsvFileSource(resources = "/two-column.csv", numLinesToSkip = 1)
void testWithCsvFileSourceFromClasspath(String country, int reference) {
    assertNotNull(country);
    assertNotEquals(0, reference);
}

@ParameterizedTest
@CsvFileSource(files = "src/test/resources/two-column.csv", numLinesToSkip = 1)
void testWithCsvFileSourceFromFile(String country, int reference) {
    assertNotNull(country);
    assertNotEquals(0, reference);
}

@ParameterizedTest(name = "[{index}] {arguments}")
@CsvFileSource(resources = "/two-column.csv", useHeadersInDisplayName = true)
void testWithCsvFileSourceAndHeaders(String country, int reference) {
    assertNotNull(country);
    assertNotEquals(0, reference);
}
```

#### 13.1.6 @ArgumentsSource

用于指定自定义的、可重用的ArgumentsProvider。必须将ArgumentsProvider的实现声明为顶级类或静态嵌套类。

```java
@ParameterizedTest
@ArgumentsSource(MyArgumentsProvider.class)
void testWithArgumentsSource(String argument) {
    assertNotNull(argument);
}
```

```java
public class MyArgumentsProvider implements ArgumentsProvider {

    @Override
    public Stream<? extends Arguments> provideArguments(ExtensionContext context) {
        return Stream.of("apple", "banana").map(Arguments::of);
    }
}
```

### 13.2 Argument Conversion（参数转换）

Junit提供了内置的隐式转换器。

| Target Type                | Example                                                                                                           |
| -------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| `boolean`/`Boolean`        | `"true"` → `true`                                                                                                 |
| `byte`/`Byte`              | `"15"`, `"0xF"`, or `"017"` → `(byte) 15`                                                                         |
| `char`/`Character`         | `"o"` → `'o'`                                                                                                     |
| `short`/`Short`            | `"15"`, `"0xF"`, or `"017"` → `(short) 15`                                                                        |
| `int`/`Integer`            | `"15"`, `"0xF"`, or `"017"` → `15`                                                                                |
| `long`/`Long`              | `"15"`, `"0xF"`, or `"017"` → `15L`                                                                               |
| `float`/`Float`            | `"1.0"` → `1.0f`                                                                                                  |
| `double`/`Double`          | `"1.0"` → `1.0d`                                                                                                  |
| `Enum` subclass            | `"SECONDS"` → `TimeUnit.SECONDS`                                                                                  |
| `java.io.File`             | `"/path/to/file"` → `new File("/path/to/file")`                                                                   |
| `java.lang.Class`          | `"java.lang.Integer"` → `java.lang.Integer.class` *(use `$` for nested classes, e.g. `"java.lang.Thread$State"`)* |
| `java.lang.Class`          | `"byte"` → `byte.class` *(primitive types are supported)*                                                         |
| `java.lang.Class`          | `"char[]"` → `char[].class` *(array types are supported)*                                                         |
| `java.math.BigDecimal`     | `"123.456e789"` → `new BigDecimal("123.456e789")`                                                                 |
| `java.math.BigInteger`     | `"1234567890123456789"` → `new BigInteger("1234567890123456789")`                                                 |
| `java.net.URI`             | `"https://junit.org/"` → `URI.create("https://junit.org/")`                                                       |
| `java.net.URL`             | `"https://junit.org/"` → `new URL("https://junit.org/")`                                                          |
| `java.nio.charset.Charset` | `"UTF-8"` → `Charset.forName("UTF-8")`                                                                            |
| `java.nio.file.Path`       | `"/path/to/file"` → `Paths.get("/path/to/file")`                                                                  |
| `java.time.Duration`       | `"PT3S"` → `Duration.ofSeconds(3)`                                                                                |
| `java.time.Instant`        | `"1970-01-01T00:00:00Z"` → `Instant.ofEpochMilli(0)`                                                              |
| `java.time.LocalDateTime`  | `"2017-03-14T12:34:56.789"` → `LocalDateTime.of(2017, 3, 14, 12, 34, 56, 789_000_000)`                            |
| `java.time.LocalDate`      | `"2017-03-14"` → `LocalDate.of(2017, 3, 14)`                                                                      |
| `java.time.LocalTime`      | `"12:34:56.789"` → `LocalTime.of(12, 34, 56, 789_000_000)`                                                        |
| `java.time.MonthDay`       | `"--03-14"` → `MonthDay.of(3, 14)`                                                                                |
| `java.time.OffsetDateTime` | `"2017-03-14T12:34:56.789Z"` → `OffsetDateTime.of(2017, 3, 14, 12, 34, 56, 789_000_000, ZoneOffset.UTC)`          |
| `java.time.OffsetTime`     | `"12:34:56.789Z"` → `OffsetTime.of(12, 34, 56, 789_000_000, ZoneOffset.UTC)`                                      |
| `java.time.Period`         | `"P2M6D"` → `Period.of(0, 2, 6)`                                                                                  |
| `java.time.YearMonth`      | `"2017-03"` → `YearMonth.of(2017, 3)`                                                                             |
| `java.time.Year`           | `"2017"` → `Year.of(2017)`                                                                                        |
| `java.time.ZonedDateTime`  | `"2017-03-14T12:34:56.789Z"` → `ZonedDateTime.of(2017, 3, 14, 12, 34, 56, 789_000_000, ZoneOffset.UTC)`           |
| `java.time.ZoneId`         | `"Europe/Berlin"` → `ZoneId.of("Europe/Berlin")`                                                                  |
| `java.time.ZoneOffset`     | `"+02:30"` → `ZoneOffset.ofHoursMinutes(2, 30)`                                                                   |
| `java.util.Currency`       | `"JPY"` → `Currency.getInstance("JPY")`                                                                           |
| `java.util.Locale`         | `"en"` → `new Locale("en")`                                                                                       |
| `java.util.UUID`           | `"d043e930-7b3b-48e3-bdbe-5a3ccfb833db"` → `UUID.fromString("d043e930-7b3b-48e3-bdbe-5a3ccfb833db")`              |

字符串到对象的转换

- 工厂方法：在目标类型中声明非静态私有方法，它接受单个 String 参数并返回目标类型的实例。 方法的名称可以是任意的，不需要遵循任何特定的约定。

- 工厂构造函数：目标类型中接受单个字符串参数的非私有构造函数。 请注意，目标类型必须声明为顶级类或静态嵌套类。

```java
@ParameterizedTest
@ValueSource(strings = "42 Cats")
void testWithImplicitFallbackArgumentConversion(Book book) {
    assertEquals("42 Cats", book.getTitle());
}
```

```java
public class Book {

    private final String title;

    private Book(String title) {
        this.title = title;
    }

    public static Book fromTitle(String title) {
        return new Book(title);
    }

    public String getTitle() {
        return this.title;
    }
}
```

显示转换

使用@ConvertWith注释显式指定 ArgumentConverter 用于某个参数，而不是依赖于隐式参数转换。 请注意，必须将ArgumentConverter的实现声明为顶级类或静态嵌套类。

```java
@ParameterizedTest
@EnumSource(ChronoUnit.class)
void testWithExplicitArgumentConversion(
        @ConvertWith(ToStringArgumentConverter.class) String argument) {

    assertNotNull(ChronoUnit.valueOf(argument));
}
```

```java
public class ToStringArgumentConverter extends SimpleArgumentConverter {

    @Override
    protected Object convert(Object source, Class<?> targetType) {
        assertEquals(String.class, targetType, "Can only convert to String");
        if (source instanceof Enum<?>) {
            return ((Enum<?>) source).name();
        }
        return String.valueOf(source);
    }
}
```

如果转换器仅用于将一种类型转换为另一种类型，则可以扩展TypedArgumentConverter。

```java
public class ToLengthArgumentConverter extends TypedArgumentConverter<String, Integer> {

    protected ToLengthArgumentConverter() {
        super(String.class, Integer.class);
    }

    @Override
    protected Integer convert(String source) {
        return (source != null ? source.length() : 0);
    }
}
```

### 13.3 Argument Aggregation（参数聚合）

默认情况下，提供给 @ParameterizedTest 方法的每个参数都对应一个方法参数。 因此，预期会提供大量参数的参数源可能会导致较大的方法签名。

在这种情况下，可以使用 ArgumentsAccessor 代替多个参数。

```java
@ParameterizedTest
@CsvSource({
    "Jane, Doe, F, 1990-05-20",
    "John, Doe, M, 1990-10-22"
})
void testWithArgumentsAccessor(ArgumentsAccessor arguments) {
    Person person = new Person(arguments.getString(0),
                               arguments.getString(1),
                               arguments.get(2, Gender.class),
                               arguments.get(3, LocalDate.class));

    if (person.getFirstName().equals("Jane")) {
        assertEquals(Gender.F, person.getGender());
    }
    else {
        assertEquals(Gender.M, person.getGender());
    }
    assertEquals("Doe", person.getLastName());
    assertEquals(1990, person.getDateOfBirth().getYear());
}
```

#### 13.3.1 Custom Aggregators

除了使用ArgumentsAccessor直接访问 @ParameterizedTest 方法的参数外，还支持使用自定义的、可重用的聚合器。

要使用自定义聚合器，需实现ArgumentsAggregato 接口并在@ParameterizedTest注解的方法中使用@AggregateWith注解参数。

```java
@ParameterizedTest
@CsvSource({
    "Jane, Doe, F, 1990-05-20",
    "John, Doe, M, 1990-10-22"
})
void testWithArgumentsAggregator(@AggregateWith(PersonAggregator.class) Person person) {
    // perform assertions against person
}
```

```java
public class PersonAggregator implements ArgumentsAggregator {
    @Override
    public Person aggregateArguments(ArgumentsAccessor arguments, ParameterContext context) {
        return new Person(arguments.getString(0),
                          arguments.getString(1),
                          arguments.get(2, Gender.class),
                          arguments.get(3, LocalDate.class));
    }
}
```

可以使用自定义组合注解避免反复申明@AggregateWith(PersonAggregator.class)。

```java
@ParameterizedTest
@CsvSource({
    "Jane, Doe, F, 1990-05-20",
    "John, Doe, M, 1990-10-22"
})
void testWithCustomAggregatorAnnotation(@CsvToPerson Person person) {
    // perform assertions against person
}
```

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.PARAMETER)
@AggregateWith(PersonAggregator.class)
public @interface CsvToPerson {
}
```

### 13.4 Customizing Dispaly Name（自定义显示名称）

```java
@DisplayName("Display name of container")
@ParameterizedTest(name = "{index} ==> the rank of ''{0}'' is {1}")
@CsvSource({ "apple, 1", "banana, 2", "'lemon, lime', 3" })
void testWithCustomDisplayNames(String fruit, int rank) {
}
```

| Placeholder            | Description                                                       |
| ---------------------- | ----------------------------------------------------------------- |
| `{displayName}`        | the display name of the method                                    |
| `{index}`              | the current invocation index (1-based)                            |
| `{arguments}`          | the complete, comma-separated arguments list                      |
| `{argumentsWithNames}` | the complete, comma-separated arguments list with parameter names |
| `{0}`, `{1}`, …​       | an individual argument                                            |

设置默认的显示名称

```properties
junit.jupiter.params.displayname.default = {index}
```

参数化方法的显示名称根据以下优先规则确定：

1.  @ParameterizedTest 的名称

2. junit.jupiter.params.displayname.default 的值

3. @ParameterizedTest 中定义的 DEFAULT_DISPLAY_NAME 常量


