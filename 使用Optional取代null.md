---
title: 使用Optional取代null
date: 2023-09-02 18:03:09
tags: [Java, Optional, null, 空指针]
---

## 使用Optional取代null

在传统的Java编程中，当方法返回一个值时，首先要判断这个返回值是否为null，只有在非空的前提下才能将其作为其他方法的参数。有时候这个值可能为空（null），当对这个空值进行操作时，就会出现空指针异常（NullPointException）。对于有一定经验的开发者而言，会在对应的地方增加null检查，如下所示：

```java
// 过多的null检查导致代码膨胀，同时代码嵌套过深，代码的可读性低
public String getCarInsuranceName(Person person) {
    if (person != null) {
        Car car = person.getCar();
        if (car != null) {
            Insurance insurance = car.getInsurance();
            if (insurance != null) {
                return insurance.getName();
            }
        }
    }
    return "Unknown";
}

// 过多的null检查导致代码膨胀，代码的可读性低
public String getCarInsuranceName(Person person) {
    if (person == null) {
        return "Unknown";
    }
    Car car = person.getCar();
    if (car == null) {
        return "Unknown";
    }
    Insurance insurance = car.getInsurance();
    if (insurance == null) {
        return "Unknown";
    }
    return insurance.getName();
} 
```

可以看到，过多的null检查带来了如下问题：

* 代码膨胀。

* 初级开发者很容易编写出嵌套过深的代码。

* 极大的降低代码的可读性。

* 过多的null检查是毫无意义的。

<!-- more -->

### Optional

`Optional`是一个容器对象，可以包含也可以不包含非null值。`Optional`是一个包装器类，其中包含对其他对象的引用。在这种情况下，对象只是指向内存位置的指针，并且也可以指向任何内容。从其它角度看，`Optional`提供一种类型级解决方案来表示可选值而不是空引用。`Optional`从如下几个方面改进了NullPointException：

* `Optional`是用来作为方法返回值的，明确表示一个值可能存在或不存在，使代码更加明确。

* 使用`Optional`强制程序员考虑空值的情况，鼓励编写更健壮、防御性的代码。

* `Optional`提供了一系列方法，使得在处理可能为空的值时更容易，避免了手动进行null检查和条件语句。

使用`Optional`重写上述代码如下所示：

```java
// 使用Optional重写
public String getCarInsuranceName(Optional<Person> person) {
    return person.flatMap(Person::getCar)
                 .flatMap(Car::getInsurance)
                 .map(Insurance::getName)
                 .orElse("Unknown");
}
```

`Optional`是 Java 中一种强大的工具，用于处理可能为空的值，提高了代码的健壮性和可读性，同时减少了空指针异常的发生。但它应该谨慎使用，只在需要的情况下使用，避免滥用：

* 变量的声明尽量少使用`Optional`，不要滥用`Optional`API。

    ```java
    String finalStatus = Optional.ofNullable(status).orElse("PENDING");
    ```

* 不要使用`Optional`作为Java Bean实例域的类型/不要使用`Optional`作为类构造器参数/不要使用`Optional`作为Java Bean Setter方法的参数。`Optional`没有实现Serializable接口（不可序列化）。

    ```java
    public class Person {
        private Optional<Car> car;
        public Optional<Car> getCar() { return car; }
    }
    ```

* 尽可能少使用`Optional`作为方法参数类型。

    ```java
    public void renderCustomer(Optional<Cart> cart, Optional<Renderer> renderer, Optional<String> name) {   
        // do something
    }
    ```

* 不要使用`Optional`作为集合的元素类型。

    ```java
    List<Optional<String>> list = new ArrayList<>(); // bad
    ```

* 不要把容器类型（包括`List`, `Set`, `Map`, `Array`, `Stream`甚至`Optional`）包装在`Optional`中。

    ```java
    Optional<List<String>> list = Optional.of(new ArrayList<>()); // bad
    ```

* 不要给`Optional`变量赋值`null`。

    ```java
    Optional<String> name = null; // bad
    Optional<Cart> emptyCart = Optional.empty(); // good
    ```

* 确保`Optional`内有值才能调用`get()`方法。

    ```java
    Optional<String> name = Optional.of("John");
    if (name.isPresent()) {
        name.get(); // good
    }else {
        // do something
    }
    Optional<String> emptyName = Optional.empty();
    emptyName.get(); // bad
    ```

* 尽量使用`Optional`提供的快捷API避免手写`if-else`语句。

    ```java
    Optional<String> name = Optional.of("John");
    name.ifPresent(System.out::println);
    ```

### Optional使用

`Optional`提供了一系列方法来处理可能为空的值，大致可分为如下几类：

#### 创建Optional对象

* `empty()`：创建一个空的`Optional`对象，表示值不存在。

    ```java
    Optional<Car> optCar = Optional.empty();
    ```

* `of(T value)`：创建一个`Optional`对象，`value`必须非空，否则会抛出`NullPointerException`。

    ```java
    Optional<Car> optCar = Optional.of(car);
    ```

* `ofNullable(T value)`：创建一个包含值或空的`Optional`对象。如果参数值为null，则返回一个空的`Optional`，否则返回包含参数值的`Optional`。

    ```java
    Optional<String> optional = Optional.ofNullable(null); // 空的 Optional
    Optional<String> optional2 = Optional.ofNullable("Hello"); // 包含值的 Optional
    ```

#### 检查Optional对象值是否存在

* `isPresent()`：如果值存在则返回`true`，否则返回`false`。

    ```java
    Optional<String> optional = Optional.ofNullable(null);
    optional.isPresent(); // false
    ```

#### 执行操作（如果值存在）

* `ifPresent(Consumer<? super T> consumer)`：如果值存在则执行`consumer`，否则不执行任何操作。

    ```java
    Optional<String> optional = Optional.ofNullable("Hello");
    optional.ifPresent(System.out::println); // Hello
    ```

#### 获取Optional对象值或默认值

* `get()`：如果值存在则返回值，否则抛出`NoSuchElementException`。

    ```java
    Optional<String> optional = Optional.ofNullable("Hello");
    optional.get(); // Hello
    ```

* `orElse(T other)`：如果值存在则返回值，否则返回`other`。

    ```java
    Optional<String> optional = Optional.ofNullable(null);
    optional.orElse("World"); // World
    ```

* `orElseGet(Supplier<? extends T> other)`：如果值存在则返回值，否则返回`Supplier`提供的值。

    ```java
    Optional<String> optional = Optional.ofNullable(null);
    optional.orElseGet(() -> "World"); // World
    ```

* `orElseThrow(Supplier<? extends X> exceptionSupplier)`：如果值存在则返回值，否则抛出`exceptionSupplier`提供的异常。

    ```java
    Optional<String> optional = Optional.ofNullable(null);
    optional.orElseThrow(() -> new RuntimeException("Value not present")); // 抛出异常
    ```

#### 转换和过滤Optional对象值

* `map(Function<? super T, ? extends U> mapper)`：如果值存在则对其执行`mapper`函数调用，返回一个`Optional`对象，否则返回一个空的`Optional`对象。

    ```java
    Optional<String> optional = Optional.ofNullable("Hello");
    optional.map(String::toUpperCase); // HELLO
    ```

* `flatMap(Function<? super T, Optional<U>> mapper)`：如果值存在则对其执行`mapper`函数调用，返回一个`Optional`对象，否则返回一个空的`Optional`对象。（类似于流的扁平化映射）

    ```java
    Optional<String> optional = Optional.ofNullable("Hello");
    optional.flatMap(s -> Optional.ofNullable(s.toUpperCase())); // HELLO
    ```

* `filter(Predicate<? super T> predicate)`：如果值存在并且满足`predicate`条件则返回包含该值的`Optional`对象，否则返回一个空的`Optional`对象。

    ```java
    Optional<String> optional = Optional.ofNullable("Hello");
    optional.filter(s -> s.length() > 5); // Optional.empty
    ```
