---
title: 如何编写Java单元测试
date: 2024-02-17 14:05:58
tags: 单元测试、Java、JUnit4、JUnit5、Mockito、PowerMock、Spock
---

## 什么是单元测试

单元测试是一种软件测试方法，它针对软件代码的单个单元进行测试，确保该单元代码在特定输入下能够产生预期的输出。单元测试是面向对象编程的重要组成部分，也是测试驱动开发 (TDD) 的基础。

在软件开发中，单元测试是一种重要的质量保证手段。通过编写和执行单元测试，开发人员可以对代码进行验证，确保其功能正确、可靠、稳定。

## 单元测试的必要性

单元测试的必要性体现在如下几个方面：

* **验证代码逻辑正确性**：单元测试可以验证代码中的各个单元（函数、方法、类）是否按照预期工作，从而保证代码的逻辑正确性。

* **提高代码质量**：通过编写单元测试，可以发现并解决潜在的代码缺陷和逻辑错误，从而提高代码的质量。

* **支持重构和修改**：单元测试可以提供一种安全的修改代码的方式，通过保证单元测试的通过性，开发人员可以放心地进行重构和修改，而不会破坏原有功能。

* **提高开发效率**：单元测试可以自动化执行，减少手动测试的时间，提高开发效率，同时还能够快速反馈代码修改对系统功能的影响。

## 单元测试的编写原则

编写单元测试时需要确保编写的单元测试具有良好的质量、可维护性和可靠性。以下是一些常见的单元测试编写原则：

* **独立性（Independence）**：每个单元测试应该是相互独立的，不应该依赖于其他测试的执行顺序或结果。这样可以确保每个测试都能独立地验证被测试代码的功能。

* **可重复性（Repeatability）**：单元测试应该能够在任何环境下重复执行，并且始终产生相同的结果。这样可以确保测试结果的可靠性和稳定性。

* **可读性（Readability）**：单元测试应该易于阅读和理解，以便开发人员可以快速了解测试的目的和逻辑。采用清晰的命名、注释和结构化的代码可以提高测试的可读性。

* **简单性（Simplicity）**：单元测试应该保持简单，避免过于复杂的逻辑和结构。每个测试应该专注于验证特定功能或行为，保持测试方法的简洁性和直观性。

* **快速性（Speed）**：单元测试应该能够快速执行，以便频繁运行和快速反馈。快速的单元测试可以促进开发流程的迭代和持续集成。

* **全面性（Comprehensiveness）**：单元测试应该覆盖代码的各个功能和路径，确保被测试代码的各个方面都得到验证。通过全面的测试覆盖可以提高测试的可靠性和有效性。

* **稳定性（Stability）**：单元测试应该是稳定的，不受外部环境和条件的影响。稳定的测试可以减少误报和误导，确保测试结果的准确性。

* **维护性（Maintainability）**：单元测试应该易于维护，随着代码的变化和演进而进行更新和调整。采用模块化、可重用的测试代码和合适的设计模式可以提高测试的可维护性。

<!-- more -->

## Java单元测试框架

在Java开发中，有多种单元测试框架可供选择，其中最常用的包括JUnit、TestNG、Mockito、PowerMock、Spock等。下面将介绍其中几种常用的Java单元测试框架。

### JUnit4

JUnit4是Java中最流行的单元测试框架之一，提供了一组注解和断言方法，可以轻松地编写和执行单元测试。

JUnit4的单元测试编写主要包括以下步骤：

* 使用`@Test`注解标识测试方法。

* 使用断言方法（如`assertEquals()`、`assertTrue()`）验证方法调用的结果。

如下所示：

```java
import static org.junit.Assert.assertEquals;

public class JUnit4CalculateTest {

    @Test
    public void testAdd() {
        JUnit4Calculate jUnit4Calculate = new JUnit4Calculate();
        int result = jUnit4Calculate.add(1, 2);
        assertEquals(3, result);
    }
}
```

#### JUnit4注解

JUnit4通过注解的方式来识别测试方法。目前支持的主要注解如下：

* `@BeforeClass`：用于标记一个测试类中的静态方法，这个方法会在所有测试方法之前执行，全局只会执行一次，而且是第一个运行，在一个类中，只能有一个`@BeforeClass`注解的方法。如果有多个，JUnit只会执行一个。这个注解通常用于执行一些只需要在类开始测试时执行一次的准备工作，比如设置静态变量，或者打开数据库连接等。

* `@AfterClass`：用于标记一个测试类中的静态方法，这个方法会在所有测试方法之后执行，全局只会执行一次，而且是最后一个运行，在一个类中，只能有一个`@AfterClass`注解的方法。如果有多个，JUnit只会执行一个。这个注解通常用于执行一些只需要在类结束测试时执行一次的清理工作，比如关闭数据库连接等。

* `@Before`：用于标记一个测试方法，这个方法会在每个测试方法之前执行，每个测试方法执行前都会执行一次。这个注解通常用于执行一些只需要在每个测试方法执行前执行的准备工作，比如初始化对象等。

* `@After`：用于标记一个测试方法，这个方法会在每个测试方法之后执行，每个测试方法执行后都会执行一次。这个注解通常用于执行一些只需要在每个测试方法执行后执行的清理工作，比如释放资源等。

* `@Test`：用于标记一个测试方法，这个方法会被JUnit执行。这个注解通常用于标记测试方法。

* `@Ignore`：用于标记一个测试方法或测试类，这个方法或类会被JUnit忽略。这个注解通常用于标记不需要执行的测试方法或测试类。

* `@RunWith`：改变测试运行器。测试运行器是一个类，它负责运行测试并报告测试结果。JUnit4包含了一些内置的运行器，如BlockJUnit4ClassRunner，Parameterized，Suite等。也可以创建自定义的测试运行器，只需要创建一个类，继承Runner类，然后实现必要的方法。

  * `BlockJUnit4ClassRunner`：这是默认的测试运行器，如果你没有使用@RunWith注解指定运行器，那么就会使用这个运行器。它支持标准的JUnit4测试类，包括`@Test`，`@Before`，`@After`，`@BeforeClass`，`@AfterClass`等注解。  

  * `Parameterized`：这个运行器用于参数化测试。参数化测试允许你使用不同的参数多次运行同一个测试。你需要使用`@Parameters`注解提供一个参数集合，然后在测试类的构造函数中接收这些参数。  

  * `Suite`：这个运行器用于组合多个测试类一起运行。你需要使用`@SuiteClasses`注解指定要运行的测试类。

* `@Parameters`：用于参数化测试。参数化测试允许开发者使用不同的参数多次运行同一测试用例，以检查不同的条件。  @Parameters注解应用于返回集合的公共静态方法上，该集合包含为测试方法提供参数的数组。每个数组的元素数量应与参数化测试方法的参数数量相匹配。

* `@SuiteClasses`：用于指定一组测试类，这些测试类会被一起运行。这个注解通常用于组合多个测试类一起运行。

#### JUnit4断言

JUnit4提供了一组断言方法，用于验证方法调用的结果是否符合预期。常用的断言方法包括：

* `assertEquals()`：验证两个值是否相等。

* `assertNotEquals()`：验证两个值是否不相等。

* `assertTrue()`：验证条件是否为真。

* `assertFalse()`：验证条件是否为假。

* `assertNull()`：验证值是否为null。

* `assertNotNull()`：验证值是否不为null。

* `assertArrayEquals()`：验证两个数组是否相等。

* `assertSame()`：验证两个对象引用是否指向同一个对象。

* `assertNotSame()`：验证两个对象引用是否指向不同的对象。

* `assertThat()`：接受一个匹配器（Matcher）对象作为参数，用于定义测试的预期行为。

#### JUnit4异常处理

* `assertThrows()`：断言执行某段代码时会抛出预期的异常。

* `expected`属性：通过`@Test`注解的`expected`属性来指定预期的异常类型。

* `timeout`属性：通过`@Test`注解的`timeout`属性来指定测试方法的超时时间。

#### JUnit4单测示例

```java
import org.junit.Test;
import static org.junit.Assert.*;
import org.junit.Rule;
import org.junit.rules.ExpectedException;
import org.hamcrest.CoreMatchers;

// 默认为 @RunWith(BlockJUnit4ClassRunner.class) 
// @RunWith(BlockJUnit4ClassRunner.class) 
public class CalculatorTest {

    private Calculator calculator = new Calculator();

    // @BeforeClass只能修饰静态方法，最先执行且只执行一次 
    @BeforeClass
    public static void beforeClass() {
        System.out.println("beforeClass");
    }

    // @AfterClass只能修饰静态方法，最后执行且只执行一次
    @AfterClass
    public static void afterClass() {
        System.out.println("afterClass");
    }

    // @Before在每个测试方法执行前都会执行一次
    @Before
    public void before() {
        System.out.println("before");
    }

    // @After在每个测试方法后都会执行一次
    @After
    public void after() {
        System.out.println("after");
    }


    @Test
    public void testAdd() {
        // 使用assertEquals测试结果是否符合预期
        assertEquals(5, calculator.add(2, 3));
    }

    @Test
    public void testSubtract() {
        // 使用assertNotEquals测试结果是否不符合预期
        assertNotEquals(5, calculator.subtract(3, 2));
    }

    @Test
    public void testMultiply() {
        // 使用assertTrue和assertFalse测试条件是否为真或假
        assertTrue(calculator.multiply(2, 3) == 6);
        assertFalse(calculator.multiply(2, 3) == 5);
    }

    @Test
    public void testDivide() {
        // 使用assertThat和CoreMatchers.equalTo测试结果是否符合预期
        // 接受一个匹配器（Matcher）对象作为参数，用于定义测试的预期行为。assertThat()方法的可读性更强，可以使测试的意图更加明显。
        assertThat(calculator.divide(6, 3), CoreMatchers.equalTo(2.0));
    }

    @Test(expected = ArithmeticException.class)
    public void testDivideByZero() {
        // 使用expected属性测试是否抛出预期的异常
        calculator.divide(5, 0);
    }

    @Test(timeout = 1000)
    public void testTimeout() {
        // 使用timeout属性测试方法是否在预期的时间内完成
        calculator.longRunningOperation();
    }

    @Test
    public void testDivideByZeroWithAssertThrows() {
        // 使用assertThrows测试是否抛出预期的异常，并检查异常的消息
        ArithmeticException thrown = assertThrows(
            ArithmeticException.class,
            () -> calculator.divide(5, 0)
        );
        assertEquals("/ by zero", thrown.getMessage());
    }

    // @Ignore标记不需要执行的测试方法
    @Test
    @Ignore
    public void testIgnore() {
        calculator.longRunningOperation();
    }
}
```

使用JUnit4进行参数化测试，创建参数化测试需要遵循五个步骤

* 使用@RunWith（Parameterized.class）注释测试类。

* 创建一个使用@Parameters注释的公共静态方法，该方法返回一个对象集合（Collection）作为测试数据集，Collection的具体类型并不重要，重要的是它包含的元素。每个元素都应该是一个数组，这个数组的元素将作为参数传递给参数化测试的构造函数和测试方法。

* 创建一个公共构造函数，它接受相当于一行“测试数据”的内容。

* 为测试数据的每个“列”创建一个实例变量。

* 使用实例变量作为测试数据的来源创建测试用例。

```java
package org.carey.junit4;

import org.junit.*;
import org.junit.runner.RunWith;
import org.junit.runners.Parameterized;

import java.util.Arrays;
import java.util.Collection;

import static org.junit.Assert.assertEquals;

@RunWith(Parameterized.class)
public class JUnit4CalculateTest {

    private int input1;
    private int input2;
    private int expected;

    public JUnit4CalculateTest(int input1, int input2, int expected) {
        this.input1 = input1;
        this.input2 = input2;
        this.expected = expected;
    }

    @Parameters
    public static Collection<Object[]> data() {
        return Arrays.asList(new Object[][]{
                {1, 2, 3},
                {2, 3, 5},
                {3, 4, 7},
                {4, 5, 9},
                {5, 6, 11}
        });
    }

    @Test
    public void testAdd() {
        JUnit4Calculate jUnit4Calculate = new JUnit4Calculate();
        int result = jUnit4Calculate.add(input1, input2);
        assertEquals(expected, result);
    }
}
```

使用JUnit4进行套件测试如下：

```java
public class TestClass1 {
    @Test
    public void test1() {
        System.out.println("TestClass1: test1");
    }

    @Test
    public void test2() {
        System.out.println("TestClass1: test2");
    }
}

public class TestClass2 {
    @Test
    public void test1() {
        System.out.println("TestClass2: test1");
    }

    @Test
    public void test2() {
        System.out.println("TestClass2: test2");
    }
}
```

```java
import org.junit.runner.RunWith;
import org.junit.runners.Suite;

@RunWith(Suite.class)
@Suite.SuiteClasses({
    TestClass1.class,
    TestClass2.class
})
public class TestSuite {
    // 这个类不需要包含任何代码
}
```
